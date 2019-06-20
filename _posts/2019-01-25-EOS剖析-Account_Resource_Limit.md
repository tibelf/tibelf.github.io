# EOS 剖析（二）
这一周期，我们来分析下 账户资源限额（Account Resource Limit）。

EOS 提出的概念就是用户在区块链上的所有操作应该是免费的，如果不是免费的，那么就不会有人来玩；

可以类比在现实中，你用微信进行沟通，只要有网络，那么就可以互相发信息，而且是完全免费的才对，如果每发一条信息，就要扣除一定的手续费，如此的费用开销太大了。

为了实现这个目的，相对于大家熟知的 BTC 和 ETH，EOS 提出了 CPU，Memory，RAM 三个新概念

* ***CPU***：执行交易的时候需要耗费，以毫秒为单位；
* ***Memory***：进行交易的内容传输时候需要耗费，以字节为单位；
* ***RAM***：存储账号的相关信息需要消耗，以字节为单位；

CPU 和 Memory 都可以通过 EOS token 通过质押的方式获得，属于消耗品，但是有一个重生周期。RAM 需要通过 EOS token 去购买，也属于消耗品，但是没有重生周期，只能重新购买才能获取。

可能有人会问，如果我不想玩了，质押的 EOS token 是否可以还给我呢？答案是肯定的，只要你发起退款命令即可，系统自动会在 3 天后把你质押的 EOS token 返回到你的账户中。

可能还有人会问，那 RAM 我如果没用完怎么办？答案是你可以卖给别人。

## 账户充值/退款流程图
![流程图](https://raw.githubusercontent.com/tibelf/image/master/image/account_resource_limit.jpeg)

* EOS 有一个官方的客户端：cleos，流程图中的 main.cpp 是在代码路径 `/eos/programs/cleos/` 下；

## changebw() 代码分析
```
void system_contract::changebw( account_name from, account_name receiver,
                                   const asset stake_net_delta, const asset stake_cpu_delta, bool transfer )
   {
      require_auth( from );
      eosio_assert( stake_net_delta != asset(0) || stake_cpu_delta != asset(0), "should stake non-zero amount" );
      eosio_assert( std::abs( (stake_net_delta + stake_cpu_delta).amount )
                     >= std::max( std::abs( stake_net_delta.amount ), std::abs( stake_cpu_delta.amount ) ),
                    "net and cpu deltas cannot be opposite signs" );

      account_name source_stake_from = from;
      
      // 默认值为 false，如果值为 true，代表 CPU，NET，vote 权限都给了 receiver
      if ( transfer ) {
         from = receiver;
      }

      // update stake delegated from "from" to "receiver"
      // 更新 del_bandwidth_table 表信息（from 是 key），里面存储的是由 from 发起的 stake token 的记录信息，详细记录了从 from 中 stake token 来换取 CPU 和 NET 给 reciver 的操作（ from 和 receiver 可以是同一个人，也可以是不同的，所以 del_bandwidth_table 表中可能会存在不同的 from 到同一个 receiver 的记录信息）
      {
         del_bandwidth_table     del_tbl( _self, from);
         auto itr = del_tbl.find( receiver );
         if( itr == del_tbl.end() ) {
            itr = del_tbl.emplace( from, [&]( auto& dbo ){
                  dbo.from          = from;
                  dbo.to            = receiver;
                  dbo.net_weight    = stake_net_delta;
                  dbo.cpu_weight    = stake_cpu_delta;
               });
         }
         else {
            del_tbl.modify( itr, 0, [&]( auto& dbo ){
                  dbo.net_weight    += stake_net_delta;
                  dbo.cpu_weight    += stake_cpu_delta;
               });
         }
         eosio_assert( asset(0) <= itr->net_weight, "insufficient staked net bandwidth" );
         eosio_assert( asset(0) <= itr->cpu_weight, "insufficient staked cpu bandwidth" );
         // 执行 undelegate 操作会有可能触发
         if ( itr->net_weight == asset(0) && itr->cpu_weight == asset(0) ) {
            del_tbl.erase( itr );
         }
      } // itr can be invalid, should go out of scope

      // update totals of "receiver"
      // 更新 user_resources_table 表信息（receiver 是 key）
      {
         user_resources_table   totals_tbl( _self, receiver );
         auto tot_itr = totals_tbl.find( receiver );
         if( tot_itr ==  totals_tbl.end() ) {
            tot_itr = totals_tbl.emplace( from, [&]( auto& tot ) {
                  tot.owner = receiver;
                  tot.net_weight    = stake_net_delta;
                  tot.cpu_weight    = stake_cpu_delta;
               });
         } else {
            totals_tbl.modify( tot_itr, from == receiver ? from : 0, [&]( auto& tot ) {
                  tot.net_weight    += stake_net_delta;
                  tot.cpu_weight    += stake_cpu_delta;
               });
         }
         eosio_assert( asset(0) <= tot_itr->net_weight, "insufficient staked total net bandwidth" );
         eosio_assert( asset(0) <= tot_itr->cpu_weight, "insufficient staked total cpu bandwidth" );

         // 更新 receiver 通过 stake token 获得的可用 CPU 和 NET 上限值
         set_resource_limits( receiver, tot_itr->ram_bytes, tot_itr->net_weight.amount, tot_itr->cpu_weight.amount );
        
        // 执行 undelegate 操作会有可能触发
         if ( tot_itr->net_weight == asset(0) && tot_itr->cpu_weight == asset(0)  && tot_itr->ram_bytes == 0 ) {
            totals_tbl.erase( tot_itr );
         }
      } // tot_itr can be invalid, should go out of scope

      // create refund or update from existing refund
      // 更新 refund_table 表信息（from 是 key），delegate 和 undelegate 同时考虑
      // 如果发起的是 delegate 操作，并且从 refund_table 中找到对应 from 信息，优先把从 undelegate 更新需要 delegate 的 token。如果 undelegate 的对应 CPU 和 NET 都为 0 了，那么将原有的 unstake trx 任务删除，如果 undelegate 的对应 CPU 和 NET 的值有更新，那么并将原有的 unstake trx 任务删除，重新发起一个新的 unstake trx 操作。
      // 如果发起的是 undelegate 操作，并且从 refund_table 中找到对应 from 信息，更新 refund_table 中对应 from 的 CPU 和 NET 值，并将原有的 unstake trx 任务删除，重新发起一个新的 unstake trx 操作；如果未找到，在 refund_table 中插入一条新记录，并发起一个 unstake token 的 trx，设置延迟执行的时间为 3 天后。
      if ( N(eosio.stake) != source_stake_from ) { //for eosio both transfer and refund make no sense
         refunds_table refunds_tbl( _self, from );
         auto req = refunds_tbl.find( from );

         //create/update/delete refund
         auto net_balance = stake_net_delta;
         auto cpu_balance = stake_cpu_delta;
         bool need_deferred_trx = false;


         // net and cpu are same sign by assertions in delegatebw and undelegatebw
         // redundant assertion also at start of changebw to protect against misuse of changebw
         bool is_undelegating = (net_balance.amount + cpu_balance.amount ) < 0;
         bool is_delegating_to_self = (!transfer && from == receiver);

         if( is_delegating_to_self || is_undelegating ) {
            if ( req != refunds_tbl.end() ) { //need to update refund
               refunds_tbl.modify( req, 0, [&]( refund_request& r ) {
                  if ( net_balance < asset(0) || cpu_balance < asset(0) ) {
                     r.request_time = now();
                  }
                  r.net_amount -= net_balance;
                  if ( r.net_amount < asset(0) ) {
                     net_balance = -r.net_amount;
                     r.net_amount = asset(0);
                  } else {
                     net_balance = asset(0);
                  }
                  r.cpu_amount -= cpu_balance;
                  if ( r.cpu_amount < asset(0) ){
                     cpu_balance = -r.cpu_amount;
                     r.cpu_amount = asset(0);
                  } else {
                     cpu_balance = asset(0);
                  }
               });

               eosio_assert( asset(0) <= req->net_amount, "negative net refund amount" ); //should never happen
               eosio_assert( asset(0) <= req->cpu_amount, "negative cpu refund amount" ); //should never happen

               if ( req->net_amount == asset(0) && req->cpu_amount == asset(0) ) {
                  refunds_tbl.erase( req );
                  need_deferred_trx = false;
               } else {
                  need_deferred_trx = true;
               }

            } else if ( net_balance < asset(0) || cpu_balance < asset(0) ) { //need to create refund
               refunds_tbl.emplace( from, [&]( refund_request& r ) {
                  r.owner = from;
                  if ( net_balance < asset(0) ) {
                     r.net_amount = -net_balance;
                     net_balance = asset(0);
                  } // else r.net_amount = 0 by default constructor
                  if ( cpu_balance < asset(0) ) {
                     r.cpu_amount = -cpu_balance;
                     cpu_balance = asset(0);
                  } // else r.cpu_amount = 0 by default constructor
                  r.request_time = now();
               });
               need_deferred_trx = true;
            } // else stake increase requested with no existing row in refunds_tbl -> nothing to do with refunds_tbl
         } /// end if is_delegating_to_self || is_undelegating

         if ( need_deferred_trx ) {
            eosio::transaction out;
            out.actions.emplace_back( permission_level{ from, N(active) }, _self, N(refund), from );
            out.delay_sec = refund_delay;
            cancel_deferred( from ); // TODO: Remove this line when replacing deferred trxs is fixed
            out.send( from, from, true );
         } else {
            cancel_deferred( from );
         }

         // 如果需要 stake 的 token 数大于 0，那么需要发起一个 stake token 的 trx。
         auto transfer_amount = net_balance + cpu_balance;
         if ( asset(0) < transfer_amount ) {
            INLINE_ACTION_SENDER(eosio::token, transfer)( N(eosio.token), {source_stake_from, N(active)},
               { source_stake_from, N(eosio.stake), asset(transfer_amount), std::string("stake bandwidth") } );
         }
      }

      // update voting power
      // 更新 from 节点的投票权益，质押越多的 token，那么可以投的票数也就越多（但是投票的人数是固定的，最多不能超过 30 个BP）。有一个要注意的地方，通过 stake token 获得的 CPU 和 NET 可以转赠给 receiver，其投票权益同样可以转赠给 receiver，由 transfer 字段进行控制台，参见上面的代码。
      {
         asset total_update = stake_net_delta + stake_cpu_delta;
         auto from_voter = _voters.find(from);
         if( from_voter == _voters.end() ) {
            from_voter = _voters.emplace( from, [&]( auto& v ) {
                  v.owner  = from;
                  v.staked = total_update.amount;
               });
         } else {
            _voters.modify( from_voter, 0, [&]( auto& v ) {
                  v.staked += total_update.amount;
               });
         }
         eosio_assert( 0 <= from_voter->staked, "stake for voting cannot be negative");
         if( from == N(b1) ) {
            validate_b1_vesting( from_voter->staked );
         }

         if( from_voter->producers.size() || from_voter->proxy ) {
            update_votes( from, from_voter->proxy, from_voter->producers, false );
         }
      }
   }
```