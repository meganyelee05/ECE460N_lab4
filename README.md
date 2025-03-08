# ECE460N_lab4

## State Diagram Changes:

![App Screenshot](https://github.com/user-attachments/assets/a918eb18-f776-4a12-95c7-041638748f0a)

States 8, 48, 50, 44, 46, 37, and 26 implement the RTI instruction. They pop PC and PSR off the system stack and return to the state before the context switch. States 10, 11, 49, 52, 54, and 47 prepare the proper INTV or EXCV as well as preparing the system for a context switch; states 51, 59, 53, 55, 56, 58, 60, and 62 finish pushing PC and PSR onto the stack. States 36, 41, 34, and 39 check for access violation and unaligned access exceptions. States 2, 6, 7, and 3 now load their J-bits into TEMP, which decides the next state after state 38.  

## Datapath Changes

![App Screenshot](https://github.com/user-attachments/assets/7f68cbab-a093-4179-beb6-d375b7d64f32)

### Structures Added 
A new TEMP register stores the J-bits so that the LD/ST instructions can share exception checkers. PCMUX now has a fourth input, MDR. MDRMUX selects between loading MIOMUX or PC-2 into the MDR to minimize states and avoid the bus during the interrupt/exception handling. New registers are added for the user stack pointer and system stack pointer. A MUX selects between the two, as well as SP+2 and SP-2. Two new registers are made to store the resultant INTV/EXCV (8 bits each) and a mux selects between the different vectors. The result is left shifted and stored into Vector. UA stores whether or not there is an unaligned access exception. UAMUX selects between loading UA with logic from the bus or 0. Priv stores the privilege mode, and is loaded from PSRMUX. ACV stores whether or not there is an access violation exception. Another mux loads CC with either logic or bits [2:0] from the bus.
### Control Signals Added
LD.TEMP loads the TEMP register with the J-bits. GateTEMP gates the contents of the TEMP register onto the bus. GatePCMDR isn't needed but I was too lazy to change it. It gates MDR to PCMUX. MDRMUX selects between PC-2 and MIOMUX. SPMUX selects between the saved USP, saved SSP, SP+2, and SP-2. GateSP gates this value onto the bus. VectorMUX selects between the different vectors using logic and control signals. 00 selects the INTV (0x01) and 11 selects the unused opcode vector (0x04) but 01 and 10 will first test if ACV is asserted. 0x02 is selected if yes and 0x03 if not. LD.Vector loads both Table and Vector registers. GateVector gates a concatenated Table'Vector onto the bus. LD.UA loads UA with UAMUX. LD.Priv loads Priv from PSRMUX. PSRMUX selects from 0 or bit 15 from the bus. LD.ACV loads ACV with logic from the bus and Priv. PSRMUX also selects between logic and bits [2:0] from the bus. GatePSR gates Priv as bit 15 and CC as bits [2:0] onto the bus. DRMUX and SR1MUX now both have three select lines. 10 selects for SP (R6) and 11 is unused.

## Microsequencer Changes

![App Screenshot](https://github.com/user-attachments/assets/8b8bf9d4-53ea-40bc-a1b5-689acdab6cd7)

IRD is now two bits. 10 selects for TEMP, which means the next state is determined by the contents of TEMP (TEMP holds the J-bits from a previous state). A third COND bit, COND2, was added to select between different conditions. COND = 0 through 3 are unchanged. COND = 4 goes through when PSR[15] is asserted. COND = 5 goes through when there is an access violation. This affects J[3], J[2], and J[1] because of the state numbering requirements. COND = 6 goes through if there is an interrupt, and COND = 7 goes through if there is an unaligned access exception. UA and INT are grouped because they are never asserted at the same time and therefore don't interfere with one another. The same goes for ACV and PSR[15]. ACV, UA, and INT CONDs allow for the next state to go to the interrupt/exception handler. 
