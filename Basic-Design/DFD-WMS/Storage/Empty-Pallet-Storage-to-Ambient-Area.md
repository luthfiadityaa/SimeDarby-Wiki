[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Empty Pallet Storage database flow</span>
##<span style="color:Green; font-weight:bold">Abbreviation</span>
| CODE | TABLE NAME       |
|------|------------------|
| STRP | DNSTORAGEPLAN    | 
| PLLT | DNPALLET         | 
| WRKI | DNWORKINFO       | 
| WRKL | DNWORKLIST       | 
| CRYI | DNCARRYINFO      | 
| STCK | DNSTOCK          | 
| STCH | DNSTOCKHISTORY   |
| HSTS | DNHOSTSEND       |
| MTMS | DMMASTERMATERIAL |
| STSN | DMSTATION        |


##<span style="color:Green; font-weight:bold">Inbound Table Data Flow </span>
|Action Name| STRP | PLLT | WRKI | WRKL | CRYI | STCK | HSTS | MTMS | STSN |
|-----------|--|--|--|--|--|--|--|--|--|
| [Empty Pallet - Set (F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/Planned-Storage-to-Ambient-Area-(PKG)?_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eplanned-storage---set-(f2)%3C/span%3E) | UPDATE| | | | | | | SELECT | SELECT |

#<span style="color:skyblue; font-weight:bold">Empty Pallet - Set (F2)</span>
![image.png](/.attachments/image-ae0da33c-5f36-45e7-ad8d-527f6a9e9325.png)
The Empty Pallet 








#User Story
- [DFD Storage Stacked Empty Pallet](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5787)

# Related DFD
  - {}
