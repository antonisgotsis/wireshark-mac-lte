# Dissect SL-SCH packets with MAC-LTE

## Configuration
* Fixed fields
  * Set radioType to ```1``` (FDD)
  * Set direction to ```0``` (UL, dummy)
  * Set rntiType to ```8``` (SL-RNTI)
* Optional fields (given in hex format)
  * RNTI   : ```0065``` (dummy value)
  * UE ID  : ```04D2``` (dummy value)
  * SFN/SF : ```0661``` (dummy value)
* Data
  * **Case 1: Padding subheader placed before MAC PDU Subheader**
	* SL-SCH Subheader
	  * byte 0: ```0001 0 0 0 0``` (V/R/R/R/R)
	  * byte 1-3: ```0000 0000 0000 0000 0000 0001``` (24-bit ProSe UEID)
	  * byte 4-5: ```0000 0000 0010 0100``` (16 bit Dest Layer-2 ID)
	* Padding Subheader --> indicates padding
	  * byte 0: ```0 0 1 11111``` (R/R/E/LCID)
	* MAC PDU Subheader
	  * byte 0: ```0 0 0 01010``` (R/R/E/LCID) --> no need to indicate length, take the remainder
	* MAC PDU Payload
	  * Dummy 14 bytes: (in hex format) ```55 aa 55 aa 55 aa 55 55 aa 55 aa 55 aa 55```
  * **Case 2: Padding subheader placed after MAC PDU Subheader**
	* SL-SCH Subheader
	  * byte 0: ```0001 0 0 0 0``` (V/R/R/R/R)
	  * byte 1-3: ```0000 0000 0000 0000 0000 0001``` (24-bit ProSe UEID)
	  * byte 4-5: ```0000 0000 0010 0100``` (16 bit Dest Layer-2 ID)
	* MAC PDU Subheader
	  * byte 0: ```0 0 1 01010``` (R/R/E/LCID)
	  * byte 1: ```0 0000111``` (F/L) --> indicates a 7 byte payload
	* Padding Subheader --> indicates padding
	  * byte 0: ```0 0 0 11111``` (R/R/E/LCID)
	* MAC PDU Payload
	  * Dummy 14 bytes: (in hex format) ```55 aa 55 aa 55 aa 55 55 aa 55 aa 55 aa 55```

## Analysis by Wireshark
_Reference: 36.321 6.1.6 (MAC PDU SL-SCH)_

MAC-LTE SL-SCH: (SFN=102 , SF=1) UEId=1234 (Padding) (10:remainder)

**Case 1: Successful Recovery**
* MAC PDU Header (SL-SCH) (Padding) (10:remainder)  [2 subheaders]
  * Sub-header (SL-SCH)
	* 0001 .... = Version: 1
	* .... 0000 = Reserved bits: 0x0
	* Source Layer-2 ID: 0x000001
	* Destination Layer-2 ID: 0x0024
  * Sub-header (lcid=Padding)
	* 00.. .... = Reserved bits: 0x0
	* ..1. .... = Extension: 0x1
	* ...1 1111 = LCID: Padding (0x1f)
  * Sub-header (lcid=10, length is remainder)
	* 00.. .... = Reserved bits: 0x0
	* ..0. .... = Extension: 0x0
	* ...0 1010 = LCID: 10 (0x0a)
  * SDU (10, length=14 bytes): 55aa55aa55aa5555aa55aa55aa55

**Case 2: Failed Recovery**
* MAC PDU Header (SL-SCH)
  * Sub-header (SL-SCH)
	* 0001 .... = Version: 1
	* .... 0000 = Reserved bits: 0x0
	* Source Layer-2 ID: 0x000001
	* Destination Layer-2 ID: 0x0024
  * Sub-header (lcid=10, length is remainder)
	* 00.. .... = Reserved bits: 0x0
	* ..1. .... = Extension: 0x1
	* ...0 1010 = LCID: 10 (0x0a)
  * **[Dissector bug, protocol MAC-LTE: field mac-lte.slsch.format is not of type FT_CHAR, FT_UINT8, FT_UINT16, FT_UINT24, or FT_UINT32]**
