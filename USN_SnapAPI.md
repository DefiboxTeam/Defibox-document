# API操作示例

## 相关帐号

* 债仓管理合约: danchorsmart
* 稳定币合约: danchortoken

## 生成USN

向合约帐号 danchorsmart 转账Token, memo格式:  "`issue:质押率`" 

```
cleos transfer testuseraaaa danchorsmart "100 EOS" "issue:15000"
```

## 查询用户债仓

先查询合约数据库，获取抵押物id:

```
cleos get table danchorsmart danchorsmart collaterals

{
  "rows": [{
      "id": 1,
      "contract": "eosio.token",
      "sym": "4,EOS",
      "status": 1,
      "clear_rate": 12500,
      "forfeit": 500,
      "interest": 400,
      "min_rate": 15000,
      "last_price": 630180000,
      "min_amount": "1.0000 EOS",
      "max_amount": "10000000.0000 EOS",
      "balance": "496252.9035 EOS",
      "total_balance": "3708299.7951 EOS"
    },{
      "id": 2,
      "contract": "btc.ptokens",
      "sym": "8,PBTC",
      "status": 1,
      "clear_rate": 13000,
      "forfeit": 500,
      "interest": 400,
      "min_rate": 15000,
      "last_price": "3560823000000",
      "min_amount": "0.00010000 PBTC",
      "max_amount": "1000.00000000 PBTC",
      "balance": "0.05000000 PBTC",
      "total_balance": "0.05000000 PBTC"
    }
  ],
  "more": false
}
```


查询用户某个抵押物的债仓记录:

table name: danchorsmart, scope: 抵押物id, key: 用户账号

```
cleos get table -L testuseraaaa -U testuseraaaa danchorsmart 1 debts
```


## 通过降低质押率，生成新的USN

先查询合约数据库，获取抵押物id:

调用合约帐号的 adjust action: [用户账号，抵押物id，质押率，是否生成USN]

```
cleos push action danchorsmart adjust '["testuseraaaa", 1, 15000, true]' -p testuseraaaa
```

## 偿还USN，并减少抵押量

先查询合约数据库，获取抵押物id

向合约帐号 danchorsmart 转账USN, memo格式:  "`repay:抵押物id:质押率`" 

```
cleos transfer -c danchortoken testuseraaaa danchorsmart "11.0000 USN" "repay:1:15000"
```

## 仅偿还USN， 但不减少抵押量

先查询合约数据库，获取抵押物id

向合约帐号 danchorsmart 转账USN, memo格式:  "`repay:抵押物id:0`" 

```
cleos transfer -c danchortoken testuseraaaa danchorsmart "11.0000 USN" "repay:1:0"
```


## 仅偿还USN， 但不减少抵押量  (用抵押物支付利息)

先查询合约数据库，获取抵押物id

向合约帐号 danchorsmart 转账USN, memo格式:  "`repay2:抵押物id:0`" 

```
cleos transfer -c danchortoken testuseraaaa danchorsmart "11.0000 USN" "repay2:1:0"
```

## 仅增加抵押量

向合约帐号 danchorsmart 转账EOS, memo格式:  "`deposit`" 

```
cleos transfer testuseraaaa danchorsmart "100 EOS" "deposit"
```

## 爆仓抢拍

先查询合约数据库，获取爆仓单信息:

```
cleos get table danchorsmart danchorsmart auctions

{
  "rows": [{
      "aid": 1,
      "collateral_id": 1,
      "user": "testuseraaaa",
      "price": 55063,
      "pledge": "200.0000 EOS",
      "issue": "327.9066 USN",
      "remain_pledge": "169.9345 EOS",
      "remain_issue": "327.9066 USN",
      "create_time": "2021-05-28T07:56:00"
    },,{
      "aid": 2,
      "collateral_id": 2,
      "user": "testuserbbbb",
      "price": 105063,
      "pledge": "100.000000 BOX",
      "issue": "500.9300 USN",
      "remain_pledge": "60.950900 BOX",
      "remain_issue": "245.9300 USN",
      "create_time": "2021-05-28T07:56:00"
    }
  ],
  "more": false
}
```

如果有存在爆仓的订单，可以直接参与抢拍, 获得最多9折优惠的抵押物代币，

抢拍方式为 向合约帐号 danchorsmart 转账USN, memo格式:  "`bid:抵押物id:爆仓单id`" 

```
cleos transfer -c danchortoken testuseraaaa danchorsmart "100 USN" "bid:1:2"
```
