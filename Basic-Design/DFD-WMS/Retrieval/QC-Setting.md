[[_TOC_]]
[[_TOSP_]]

::: mermaid
flowchart TB

%% --- External Entities ---
User((User))
Timer((System Clock))

%% --- Processes ---
P1([1. Palletize Storage → Tempering])
P2([2. Retrieval for QC Start])
P3([3. Retrieval for Return Stock])
P4([4. Update QC Status])
P5([5. Extend Tempering Period])

%% --- Data Stores ---
D1[(Stock Data)]
D2[(Tempering Parameters)]
D3[(QC Records)]

%% --- Process 1 ---
User -->|Input: Start Date, Tempering Period| P1
P1 -->|Update: Stock Status=UU, Flags, DateStart, DateEnd| D1
Timer -->|Now() Check (>= DateEnd)| P1
P1 -->|Eligible Stock for QC Start| P2

%% --- Process 2 ---
P2 -->|Filter Stock Status=UU| D1
P2 -->|Update: Status QI, Flag Reached, QC Duration Start| D1
P2 -->|Record QC Start| D3
P2 -->|Send Stock (QI) | P3

%% --- Process 3 ---
P3 -->|Filter Stock Status=QI| D1
P3 -->|Add Qty, Update Stock Qty| D1
P3 -->|Updated Stock (QI)| P4

%% --- Process 4 ---
P4 -->|Filter Stock Status=QI| D1
P4 -->|Set QC Check Flag=Done, Status=UU, Stop QC Duration| D1
P4 -->|QC Completed Record| D3

%% --- Process 5 ---
P5 -->|Filter Stock Status=QI & QC Not Done| D1
P5 -->|Tempering Period Extended, Flag Not Reached| D1
:::