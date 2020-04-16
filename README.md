# cpm22src
Compile and boot CP/M 2.2 from original source

## CPMOVE
MOVCPM.COM contains three primary sections: 1) executable code for the program, 2) an image of CP/M, and 3) a relocation bitmap. For example, the table below documents how MOVCPM.COM for Altair CP/M 2.2 is laid out. Note that CP/M for a different machine may have a different bootloader size and/or BIOS size, and therefore, a slightly different layout within the file.

Offset in File 	In Memory 	Content
0000h 	0100h 	CPMOVE (code that “does” MOVCPM)
0701h-0702h 	0801h-0802h 	Length of “MODULE” (Bootloader + CCP + BDOS + BIOS)
0800h-087fh
0900h-097fh 	0900h-097fh
0a00h-0a7fh 	Bootloader (1st 128 bytes)
Bootloader (2nd 128 bytes)
0980h 	0a80h 	Start of CCP
1180h 	1280h 	Start of BDOS
1f80h 	2080h 	Start of BIOS
2580h 	2680h 	Start of relocation bitmap

The "Module" portion of MOVCPM (when MOVCPM is in memory) is the CP/M image written by SYSGEN to the boot tracks of the disk. In this case, the module runs from 900h to 267Fh. This puts the CP/M image in the same location and format as the SYSGEN program expects. Again, the starting address may vary slightly (e.g., 980h - A80h) and the length will vary based on the size of the BIOS for your particular CP/M.

The relocation bitmap has one bit corresponding to each byte in the module. If a bit is set, the corresponding byte must be updated based on where in memory CP/M is placed

In order to work with this relocation scheme, the CP/M image must be assembled so that the CCP starts at address 0000h. This leaves the BDOS starting at address 0800h and the BIOS starting at address 1600h. The bootloader is always assembled at address zero, however, relocation of the bootloader is still required as it references addresses corresponding to the final location of CP/M in memory. This CP/M image (the one assembled with the CCP starting at address zero) is the image in the module section of the file.

The bytes that must be relocated can be determined by assembling CP/M at two different addresses. The bytes that differ between the two object files are those bytes that must be updated for relocation. I have written a utility that runs on a PC and generates the bitmap for me. I then use a hex editor to patch the bitmap into the MOVCPM.COM file.

;
; create image in memory for SYSGEN as follows:
;    Address  Length   Source
;     0900h    0180h   bootloader (BOOT)
;     0a80h    0800h   ccp (CPM22)
;     1280h    0e00h   bdos (CPM22)
;     2080h    0880h   bios (BIOS)
;
;  Rxxxx for BOOT is always 0900h
;  Rxxxx for CPM and BIOS = 0a80h - CCP base address
;      where CCP base address = memSize - ccpSize - bdosSize - biosSize
;  For a 59K CPM, for example:
;      59*1024 - 800h - e00h - 700h = cf00h, so R = a80h-cf00h = 3b80h
;
; BOOT is split into two 080h byte sections with a 080h
;   gap in the middle so that it appears on disk in every
;   other sector.
;
DDT
F100 2900 0
IBOOT.HEX
R0900
M980 9FF A00
F980 9FF 0
ICPM22.HEX
R3b80
IBIOS.HEX
R3b80
G0
SAVE 40 CPM.COM
ERA CPM22.HEX
ERA BOOT.HEX
ERA BIOS.HEX
;************************************************************
;
; Run the SYSGEN command NOW to place this CPM image
; on the drive you choose. When prompted for the source,
; press RETURN -- this will use the CPM image just
; created in memory. Do not run any other CPM commands
; before running the SYSGEN command!
;
; NOTE: To place this CPM image on a disk in the future,
;    run SYSGEN CPM.COM
;
;************************************************************
