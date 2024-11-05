### [M-#] TITLE (Root Cause + Impact) 
- loping throught players array to check for duplicate  `PuppyRaffle:::enterRaffle` in is a Denial of Service attack, increments gas fee for future participants

**Description:** 
- The `PuppyRaffle:::enterRaffle`  function loops thrpugh the `players` array to check for duplicate however the longer the `PuppyRaffle:::players` is the more checks a new `player` will have to pay. This means gas cost for early players will be lower and those that joined late will be higher. Every additional address, will result in a additional loop.

```javascript
// @audit Dos Attach
    for (uint256 i = 0; i < players.length - 1; i++) {
                for (uint256 j = i + 1; j < players.length; j++) {
                    require(players[i] != players[j], "PuppyRaffle: Duplicate player");
                }
            }
```

**Impact:** 
- The gas cost for raffle entrants will greatly increase dicouraging later users from partaking and causing a rush at a start of a raffle
- An attacker might make the array so big, so that no one else wins and guarantees them winning


**Proof of Concept:**
- If we have 2 sets of 100 players the cost will be as such
- 1st 100 players: ~ 6252047 gas
- 2nd 100 players: ~ 18068137 gas

<details>
<summary>PoC</summary>
place the following test into `PuppyRaffleTest.t.sol`.

```javascript
    function testEnterRaffleFailWithLargeAmoutOfUser() public {
        vm.txGasPrice(1);
        uint256 playersNum = 100;
        address[] memory players = new address[](playersNum);
        for (uint256 i = 0; i < playersNum; i++) {
            players[i] = address(i);
        }
        uint256 gasStart = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee * playersNum}(players);
        uint256 gasEnd = gasleft();
        uint256 gasUsed = (gasStart - gasEnd) * tx.gasprice;
        console.log("Gas used for first 1000 player: ", gasUsed);


        address[] memory playersTwo = new address[](playersNum);
        for (uint256 i = 0; i < playersNum; i++) {
            playersTwo[i] = address(i + playersNum);
        }

        uint256 gasStartTwo = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee * playersNum}(playersTwo);
        uint256 gasEndTwo = gasleft();
        uint256 gasUsedTwo = (gasStartTwo - gasEndTwo) * tx.gasprice;
        console.log("Gas used for Second 1000 player: ", gasUsedTwo);

        assert(gasUsed < gasUsedTwo);
    }
```
</details>

**Recommended Mitigation:**  there are a few recommendations
1. consider alllwing duplicates. users can make new wallet addreses anyways
2. Consider using mapping to check for duplicates. This would allow constant time lookup of wether users had already entered.
