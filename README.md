# blockchain-hw-0

Выполнил: Подцепко Игорь Сергеевич, учебная группа M3335

## Задача 1

[MultiSigWallet.sol](https://github.com/gnosis/MultiSigWallet/blob/master/contracts/MultiSigWallet.sol) - сделать, чтобы с баланса multisig-контракта за одну транзакцию не могло бы уйти больше, чем 66 ETH

```diff
igor@acer:~/Documents/courses/blockchain/MultiSigWallet$ git diff
diff --git a/contracts/MultiSigWallet.sol b/contracts/MultiSigWallet.sol
index 6a777f2..68bd3c5 100644
--- a/contracts/MultiSigWallet.sol
+++ b/contracts/MultiSigWallet.sol
@@ -190,6 +190,7 @@ contract MultiSigWallet {
         public
         returns (uint transactionId)
     {
+        require(value <= 66 ether);
         transactionId = addTransaction(destination, value, data);
         confirmTransaction(transactionId);
     }
```

## Задача 2

[ERC20.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f2112be4d8e2b8798f789b948f2a7625b2350fe7/contracts/token/ERC20/ERC20.sol) - сделать, чтобы токен не мог бы быть transferred по субботам

```diff
igor@acer:~/Documents/courses/blockchain/openzeppelin-contracts$ git diff
diff --git a/contracts/token/ERC20/ERC20.sol b/contracts/token/ERC20/ERC20.sol
index 492e2610..7c114d6f 100644
--- a/contracts/token/ERC20/ERC20.sol
+++ b/contracts/token/ERC20/ERC20.sol
@@ -42,6 +42,17 @@ contract ERC20 is Context, IERC20, IERC20Metadata {
     string private _name;
     string private _symbol;
 
+    uint8 constant _SATURDAY = 6;
+
+    modifier todayIsNotSaturday() {
+        require(
+            _getCurrentWeekDay() != _SATURDAY,
+            "It is forbidden to perform the operation on Saturdays"
+        );
+        _;
+    }
+
+
     /**
      * @dev Sets the values for {name} and {symbol}.
      *
@@ -365,7 +376,7 @@ contract ERC20 is Context, IERC20, IERC20Metadata {
         address from,
         address to,
         uint256 amount
-    ) internal virtual {}
+    ) internal virtual todayIsNotSaturday {}
 
     /**
      * @dev Hook that is called after any transfer of tokens. This includes
@@ -386,4 +397,12 @@ contract ERC20 is Context, IERC20, IERC20Metadata {
         address to,
         uint256 amount
     ) internal virtual {}
+
+    function _getCurrentWeekDay() private view returns (uint8) {
+        return _getWeekDay(block.timestamp);
+    }
+
+    function _getWeekDay(uint256 timestamp) private pure returns (uint8) {
+        return uint8((timestamp / (1 days) + 4) % 7);
+    }
 }
```

## Задача 3

[DividendToken.sol](https://github.com/mixbytes/solidity/blob/076551041c420b355ebab40c24442ccc7be7a14a/contracts/token/DividendToken.sol) - сделать чтобы платеж в ETH принимался только специальной функцией, принимающей помимо ETH еще комментарий к платежу (bytes[32]). Простая отправка ETH в контракт запрещена

```diff
igor@acer:~/Documents/courses/blockchain/solidity$ git diff
diff --git a/contracts/token/DividendToken.sol b/contracts/token/DividendToken.sol
index 651290f..abff696 100644
--- a/contracts/token/DividendToken.sol
+++ b/contracts/token/DividendToken.sol
@@ -37,8 +37,9 @@ contract DividendToken is StandardToken, Ownable {
         }));
     }
 
-    function() external payable {
+    function payment(bytes32 /*comment*/) external payable {
         if (msg.value > 0) {
+            // process comment
             emit Deposit(msg.sender, msg.value);
             m_totalDividends = m_totalDividends.add(msg.value);
         }
```
