# Mobility Management Entity(MME)

- opensource: https://github.com/networkedsystemsIITB/NFV_LTE_EPC
- OpenEPC
- opencord: vMME Service(https://github.com/opencord/vMME)
- NEC's MME: Collaborate with MCORD (P20 http://opencord.org/wp-content/uploads/2016/03/M-CORD-March-2016.pdf)
- Learn: https://www.cisco.com/c/en/us/td/docs/wireless/asr_5000/21-1/MME/21-1-MME-Admin/b_21_1_MME_Admin_chapter_01.html or https://www.cisco.com/c/en/us/td/docs/wireless/asr_5000/21-4_N5-7/MME/21-4-MME-Admin/21-4-MME-Admin_chapter_01.pdf

## 簡介
- 在 EPC 中的 control plane
- 管理 session states, authentication, paging, mobility with 3GPP, 2G and 3G nodes, roaming, and other bearer management functions

## 產品描述
- 參與 activation/deactivation process，負責選擇 S-GW 和 UE 在初始階段的連接和 intra-LTE handover involving Core Network (CN) node relocation. 
- 為訂戶提供 P-GW 選擇以連接到PDN。
- 提供空閒模式 UE 跟踪和尋呼過程，包括重傳。
- 為UE選擇適當的 S-GW。
- 負責驗證用戶（通過與 HSS 交互）。
- 負責為 UE 生成和分配臨時身份。
- 檢查 UE 的授權駐留在服務提供商的公共陸地移動網絡（PLMN）上並實施UE漫遊限制。
- MME 是網絡中用於NAS信令的加密/完整性保護的終止點，並且處理安全密鑰管理。
- 與同一 PLMN 或不同 PLMN 中的MME通信。 S10 接口用於 MME 重定位和 MME 到 MME 信息傳輸或切換。
- MME 是網絡中用於 NAS 信令的加密/完整性保護的終止點，並且處理安全密鑰管理。

## 3GPP standard
- Non Access Stratum (NAS) signaling
- NAS signaling security
- Inter CN node signaling for mobility between 3GPP access networks (terminating S3)
- UE Reachability in ECM-IDLE state (including control and execution of paging retransmission)
- Tracking Area list management
- PDN GW and Serving GW selection
- MME selection for handover with MME change
- SGSN selection for handover to 2G or 3G 3GPP access networks
- Roaming (S6a towards home HSS)
- Authentication
- Bearer management functions including dedicated bearer establishment
- Lawful Interception of signaling traffic
- UE Reachability procedures
- Interfaces with MSC for Voice paging
- Interfaces with SGSN for interconnecting to legacy network 

## Logical Network Interfaces
- [Gn Interface](https://www.cisco.com/c/en/us/td/docs/wireless/asr_5000/21-1/MME/21-1-MME-Admin/b_21_1_MME_Admin_chapter_01.html#reference_3b153e49-eec8-4519-99a7-e9d301f44eca)
- [S1-MME Interface](https://www.cisco.com/c/en/us/td/docs/wireless/asr_5000/21-1/MME/21-1-MME-Admin/b_21_1_MME_Admin_chapter_01.html#reference_c266f25f-3f49-426f-a04b-5a0f5e586df8)
- [S3 Interface](https://www.cisco.com/c/en/us/td/docs/wireless/asr_5000/21-1/MME/21-1-MME-Admin/b_21_1_MME_Admin_chapter_01.html#reference_ec2ce9b8-4dc6-4cac-a981-9b5544b221f7)
- S6a Interface
- S10 Interface
- S11 Interface
- S13 Interface
- SBc Interface
- SGs Interface
- SLg Interface
- SLs Interface
- Sv Interface

## Features and Functionality - Base Software
