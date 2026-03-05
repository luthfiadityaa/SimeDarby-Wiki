# **Palletize LAYOUT**

![image.png](/.attachments/image-32e1e76d-8510-472e-9299-aaa8b33d063f.png)

**States:**

1. No Operations (Empty Pallet Supply)
2. Palletize Start
2.1 Supply Empty Pallet to 110* to 111*
2. Palletize Start. Empty pallet at Robot Station.
3. Palletize Start. Full Pallet Moving to Storage Station
4. Palletize END. The Last Pallet
5. Go back to No Operations

## **1.  No Operations (Empty Pallet Supply)**

![image.png](/.attachments/image-56482763-e68b-431c-84be-2d13a2934286.png)

::: mermaid
flowchart LR
    A[ST1210] -->|Supply Empty Pallet| B[ST1101]
    A[ST1210] -->|Supply Empty Pallet| C[ST1102]
    A[ST1210] -->|Supply Empty Pallet| D[ST1103]
    A[ST1210] -->|Supply Empty Pallet| E[ST1104]
    A[ST1210] -->|Supply Empty Pallet| F[ST1105]
	
:::

::: mermaid	
sequenceDiagram
    autonumber
    participant AGC as AGC System
    participant WMS as WMS Logic
    participant DNP as DB: DNPallet
    participant DNS as DB: DNStock

    Note over AGC, WMS: Event: Receive ID26 (ST110*)
    AGC->>WMS: Send ID26 + Station No + BC Data
    
    rect rgb(240, 240, 240)
        Note right of WMS: Create Pallet Record
        WMS->>DNP: INSERT (STATION_NO=ID26.station_no, BCR_DATA=ID26.BC_DATA, EMPTY_FLAG=1)
        DNP-->>WMS: Return New PALLET_ID
    end

    rect rgb(220, 235, 255)
        Note right of WMS: Create Stock Record
        WMS->>DNS: INSERT (PALLET_ID, LOCATION_NO=ID26.station_no, ITEM_CODE='DIRECT_PB')
        DNS-->>WMS: Confirm Success
    end

    WMS-->>AGC: ACK / Ready for Operation
	
:::





## **2.  Palletize Start. Empty pallet at Robot Station**

EXAMPLE using station: 1105

![image.png](/.attachments/image-65593a1b-545b-406b-86ca-3f0e73822ca6.png)

## **3.  Palletize Start. Full Pallet Moving to Storage Station**

EXAMPLE using station: 1105, Storage Station: 1115

![image.png](/.attachments/image-18ba489d-8431-4d18-ab84-edae7d28ad95.png)

## **4. Palletize END. The Last Pallet**

EXAMPLE using station: 1105, Storage Station: 1115

![image.png](/.attachments/image-b9066ca9-bc0f-450e-9d48-71df945098fe.png)

## **5. Go back to No Operations**

EXAMPLE using station: 1105

![image.png](/.attachments/image-b8df59a7-0d4f-4c1b-a5e8-7539d1b4a05f.png)







