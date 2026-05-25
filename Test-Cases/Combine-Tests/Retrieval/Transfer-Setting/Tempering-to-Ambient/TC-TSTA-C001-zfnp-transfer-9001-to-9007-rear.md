[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-TSTA-C001 — Transfer Tempering → Ambient (ZFNP, source 9001 → REAR shelf 9007)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Transfer Setting Tempering → Ambient](/Basic-Design/DFD-WMS/Retrieval/Transfer-Setting/Tempering-to-Ambient) |
| **Item Scope**   | **ZFNP only** (Soft Zone 005, 002)                                                            |
| **Source**       | Tempering shelf, aisle `9001-9006` (WH 9100, single deep)                                     |
| **Destination**  | Ambient shelf, aisle `9007-9014` (WH 9200, double deep) — **REAR booked at Set(F2)**          |
| **Route**        | Shelf (9001-9006) → ID12 → BCR 7207-7214 (HP) → ID05 → SRM 9007-9014                          |
| **Path**         | `processTransferWarehouse` books REAR shelf → RetrievalSender → ID12 → ID32 → ID33 → ID64 → ID26 @ 7207-7214 → StorageSender → ID05 → ID25 → ID64 → ID33 @ 9007-9014 |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify a ZFNP pallet is transferred from a Tempering shelf in aisle 9001 to a REAR shelf of a double-deep pair in aisle 9007. Confirm `processTransferWarehouse` books **REAR only** at Set(F2), the carry's `dest=reserved_shelf_no` at the BCR conversion (via `AsrsInboundStationOperator`), and `DoubleDeepShelfSelector` handles the final placement at ID33 @ 9007.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; source SRM 9001 ACTIVE; destination SRM 9007 ACTIVE; BCR 7207 reachable.
- Pallet **P-TSTA-001** at a Tempering shelf in aisle 9001 with soft-zone **005** or **002**, status valid for transfer.
- Aisle 9007 has **≥ 2 empty pairs** (`MIN_EMPTY_PAIR_DIFFERENT_ZONE=2`).
- `DMWareHouse.last_used_station_no` is in a state that will yield 9007 in the round-robin (or test will validate whichever aisle wins).

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-TSTA-001 (ZFNP)      |
| Source Aisle ST    | 9001                   |
| Source BCR Map     | 9001 → BCR 7101 (OP inbound only — not used for transfer) |
| BCR Hop for Transfer | 7207 (HP BCR for dest aisle 9007)                       |
| Destination Aisle  | 9007                   |
| Destination Shelf  | REAR of pair A (booked at Set F2)                       |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                                                                                                          | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open `TransferSettingSCH` (Tempering → Ambient variant). Filter ZFNP.                  | P-TSTA-001 listed.                                                                                                                                                                                                                                                                                                                                       |               |                      |
| 2  | Select P-TSTA-001 and press **Set (F2)**.                                              | `processTransferWarehouse` in RetrievalSender books REAR shelf of an empty pair in aisle 9007 (round-robin via `last_used_station_no`).<br>`DNWORKINFO`, `DNWORKLIST`, `DNCARRYINFO` inserted with `CARRY_FLAG=5 (RACK_TO_RACK)`, `DEST_STATION_NO=BCR 7207`, `RESERVED_SHELF_NO=<booked rear shelf>`.<br>Destination `DMSHELF` reserved (`STATUS_FLAG=2` Reserved). |               |                      |
| 3  | RetrievalSender → **ID12** → **ID32** → **ID33** at SRM 9001 (retrieval from rack).    | `DNCARRYINFO.CMD_STATUS` cycles `2 → 3 → 5`. Source `DMSHELF.STATUS_FLAG=0`.                                                                                                                                                                                                                                                                            |               |                      |
| 4  | Inject **ID64** at STV (pickup at 9001).                                               | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                                                                                              |               |                      |
| 5  | Pallet arrives at BCR 7207 → **ID26**.                                                 | `DNARRIVAL` inserted. `AsrsInboundStationOperator` converts carry: `dest = reserved_shelf_no` (the booked rear). `DNCARRYINFO` updated.                                                                                                                                                                                                                |               |                      |
| 6  | `StorageSender.destDetermine()` runs. With `load_size_check=ON`, `DoubleDeepShelfSelector.findEmptyShelf()` decides final shelf (rear if pair empty; front if rear OCCUPIED). | For first transfer into the pair, rear is selected (matches reserved). `DNCARRYINFO` finalized, **ID05** dispatched.                                                                                                                                                                                                                                    |               |                      |
| 7  | Inject **ID25** at 7207.                                                              | `DNCARRYINFO.CMD_STATUS=3`. `DNARRIVAL` deleted.                                                                                                                                                                                                                                                                                                          |               |                      |
| 8  | Inject **ID64** at SRM 9007 (pickup of pallet at SRM end).                              | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                                                                                              |               |                      |
| 9  | Inject **ID33** at 9007 (deposit complete at REAR shelf).                               | `DMSHELF.STATUS_FLAG=1` (Occupied) for the rear shelf.<br>`DNPALLET` updated (`CURRENT_STATION_NO=9007`, location reflects rear).<br>`DNSTOCK.AREA_NO/LOCATION_NO` updated to ambient.<br>`DNINOUTRESULT` inserted.<br>`DNWORKINFO.STATUS_FLAG=4`.<br>`DNCARRYINFO` deleted. |               |                      |
| 10 | **Front-fill check (subsequent transfer)** — repeat steps 1-9 for a second ZFNP pallet `P-TSTA-002` targeting the **same pair**. | When this second carry arrives at BCR 7207, `findEmptyShelf()` sees rear is OCCUPIED → returns **front** of the same pair. Front shelf reserved, then stored. Both rear and front of the pair are now OCCUPIED.                                                                                                                                          |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT carry_key, carry_flag, dest_station_no, reserved_shelf_no
  FROM dncarryinfo WHERE pallet_id = 'P-TSTA-001';
SELECT area_no, location_no, status_flag FROM dmshelf
  WHERE area_no = :rear_area AND location_no = :rear_loc;   -- expect status_flag=2 Reserved

-- after step 9
SELECT pallet_id, current_station_no FROM dnpallet WHERE pallet_id = 'P-TSTA-001';
SELECT area_no, location_no FROM dnstock WHERE pallet_id = 'P-TSTA-001';   -- ambient area
SELECT status_flag FROM dmshelf WHERE area_no = :rear_area AND location_no = :rear_loc;   -- expect 1 Occupied
SELECT COUNT(*) FROM dncarryinfo WHERE pallet_id = 'P-TSTA-001';   -- expect 0

-- after step 10 — second pallet uses the front
SELECT status_flag FROM dmshelf WHERE area_no = :front_area AND location_no = :front_loc;   -- expect 1 Occupied
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- Source tempering shelf released (`DMSHELF.STATUS_FLAG=0`).
- Destination REAR shelf occupied; pallet at ambient aisle 9007.
- After step 10: both REAR and FRONT of the targeted pair are occupied; framework correctly fills front naturally.
- `DNCARRYINFO`, `DNARRIVAL` rows for both transfers removed.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
