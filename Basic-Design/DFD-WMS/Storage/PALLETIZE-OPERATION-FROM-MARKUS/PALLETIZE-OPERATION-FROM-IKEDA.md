# Conditions
## Initial condition
### The first day of operation
- No Empty palelt at 1101-1105 and 1111-1115.
- Batch is not execued.
- Cartons are not on production lane conveyor.

### Begining of one day after operation day.
- Empty palelt exists at 1101-1105 and may be also in front of Robot. Because AGC automatically supplies.
- Batch is not execued.
- Cartons are not on production lane conveyor.

## Condition during operation (Regardless of Batch Start/End)
### At 1101-1105
- Empty palelt exists at 1101-1105 and may be also in front of Robot. Because AGC automatically supplies.

#### In case a batch is not executed
- DNPallet and DNSotck exist but DNStoragePlan and DNCarryInfo don't exist.

#### In case a batch is executed
- DNPallet, DNSotck, DNStoragePlan and DNCarryInfo exist.

### At 1111-1115
#### In case a batch is not executed
- No palelt exist.

#### In case a batch is executed
- Palelt with palletized carton exist.
- DNPallet, DNSotck, DNStoragePlan and DNCarryInfo exist.


# Scenarios

## Normal
### Batch start setting
- Operator starts Batch on WareNavi at first.
- Operator starts Robot with corresponding setting.
- Cartons are supplied to production lane.

### Palletizing Work
#### If Empty pallet is already at 1101-1105. 
1. DNPallet and DNStock are already created when received ID26
2. WareNavi sends ID05.
   This must be triggered by Batch Start screen. Create DNCarryinfo and kick StorageSender
#### If Empty pallet just arrives at 1101-1105. 
1. WareNavi creates DNPallet and DNStock.
2. WareNavi sends ID05.
   This can be triggered from ID26 receiving process.
#### Common process
3. Empty pallet moves to front of Robot area.
4. Robot starts palletizing work.
5. Robot completes the work and release the palelt to 1111-1115.
6. Warenavi receives ID26 with Normal Completion.
7. Warenavi creates DNWorkInfo and connects it to active DNStoragePlan and DNCarryInfo.
8. WareNavi sends ID05 for going to Aisle station.

Continue to [If Empty pallet just arrives at 1101-1105].

### Palletizing Work for last pallet
1. Empty pallet arrives at 1101-1105. 
2. WareNavi creates DNPallet and DNStock.
3. WareNavi sends ID05.
   This can be triggered from ID26 receiving process.
4. Empty pallet moves to front of Robot area.
5. Robot starts palletizing work.
#### If the number of cartons is totlly planned number.
6. Robot completes the work and releases the pallet to 1111-1115.
7. Warenavi receives ID26 with Normal Completion.
8. Warenavi creates DNWorkInfo and connects it to active DNStoragePlan and DNCarryInfo.
9. WareNavi sends ID05 for going to Aisle station.
---
10. At the same time, the next pallet arrives at 1101-1105.
11. **This pallet moves to front of Robot because Batch is not Ended on WareNavi screen.**
---
12. Operator stops batch on Robot.
13. Operator executes Batch End on WareNavi screen.
14. The pallet in front of robot remains there.

#### If the number of cartons is not planned number.
6. Robot stops work because there are not cartons from production lane.
7. Operator executes Batch End on WareNavi screen. (Status --> Waiting for last pallet) Cf. Step11
8. Operator forcibly completes the pallet.
9. Robot releases the pallet to 1111-1115.
10. Warenavi receives ID26 with Force Completion.
11. Warenavi creates DNWorkInfo and connects it to DNStoragePlan(Waiting for last pallet) and DNCarryInfo.
12. WareNavi sends ID05 for going to Aisle station.
---
13. At the same time, the next pallet arrives at 1101-1105.
14. **This pallet won't move to front of Robot because Batch is Ended on WareNavi screen.**
---
