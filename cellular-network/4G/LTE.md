# LTE (Long Term Evolution)
- 長期演進技術
- 是 3GPP 家族訂定的新一代 4G 無線行動寬頻通訊系統

- Ref:  
    - http://www.3gpp.org/technologies/keywords-acronyms/98-lte  
    - https://zh.wikipedia.org/wiki/%E9%95%B7%E6%9C%9F%E6%BC%94%E9%80%B2%E6%8A%80%E8%A1%93
    - ftp://ftp.3gpp.org/Inbox/2008_web_files/LTA_Paper.pdf
    - https://www.tutorialspoint.com/lte/index.htm

## 動機
- 確保未來 3G 系統的競爭力
- higher data rates and quality of service
- Packet Switch optimised system
- 降低成本  (CAPEX and OPEX)
- 低複雜性
- Avoid unnecessary fragmentation of technologies for paired and unpaired band operation

## 相關名詞簡介
- Next Generation Mobile Networks (NGMN)
- LTE 的網路架構 = SAE(System Architecture Evolution)

## LTE 網路架構
- 三個 components：
    - **The User Equipment (UE) = Mobile Equipment (ME)**
        - The modules of ME:
            - Mobile Termination (MT) : 處理所有 communication functions
            - Terminal Equipment (TE) : 終止 data streams
            - Universal Integrated Circuit Card (UICC) : 就是 SIM 卡 (for LTE equipments). It runs an application known as the Universal Subscriber Identity Module (USIM). USIM 可以保密隱私資訊，包含 使用者手機號碼 home network identity and security keys etc.
    - **The Evolved UMTS Terrestrial Radio Access Network (E-UTRAN).**
        - architecture:
            ![](https://www.tutorialspoint.com/lte/images/lte_e_utran.jpg)  
        - E-UTRAN 處理手機(UE)和 EPC 之間的通訊。
        - 基地台名字(the evolved base stations)叫 eNodeB 或是 eNB。
        - 一個 eNB 可以處理 UE 在一個或多個 cells 中。
        - LTE 的 UE 一次只在一個 cell 中與一個 eNB 溝通。
        - eNB 主要的功能：
            - eNB 透過使用 LTE 空中架構的 analogue and digital signal processing 功能，傳送與接收無線電給所有的 UE。
            - 控制所有手機的 low-level operation, by sending them signalling messages such as handover commands.
        - eNB interface: **S1** to EPC, **X2** to nearby eNB(主要用於切換過程中的 signalling 和 packet 轉發)
            ![](https://i.imgur.com/vHaKsjv.png)  
        - home eNB (HeNB) is a base station
    - **The Evolved Packet Core (EPC).**
        - The core network
        - architecture:
            ![](https://www.tutorialspoint.com/lte/images/lte_epc.jpg)  
        - Move to [EPC.md](EPC.md)

## E-UTRAN 和 EPC 的功能劃分
![](https://www.tutorialspoint.com/lte/images/lte_epc_eutran.jpg)  

![](https://www.tutorialspoint.com/lte/images/lte_architecture.jpg)  
From: https://www.tutorialspoint.com/lte  


## SAE(System Architecture Evolution)
- 系統架構演進
- core network architecture of 3GPP's LTE wireless communication standard
- https://en.wikipedia.org/wiki/System_Architecture_Evolution

