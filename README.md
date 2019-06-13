# okD51forth
try to build a cpp forth for adafruit samd51 itsybitsy m4
## to do
	0. rom size of at most 6 K bytes
	1. the first project -- forth script interpreter
	1.0. ram (DSTKLMT 32 cells, RSTKLMT 32 cells, TIBLMT 128 bytes, TSBLMT 128 bytes)
	1.0.0. cell_t dStk[DSTKLMT]; // data stack
	1.0.0. cell_t rStk[RSTKLMT]; // return stack
	1.0.1. unsigned char tib[TIBLMT]; // terminal input buffer
	1.0.2. unsigned char stb[TSBLMT]; // string token buffer
	1.0.3. system variables // cell_t sysCell[32]; uint8_t* sysByte = (uint8_t*) sysCell;
	1.0.3.0. sysCell[0]=0; // xtb // the xt, execute token, to boot the system // xt is iRomWord + 1
	1.0.3.0. sysCell[1]=0; // ip // inner interpreting pointer
	1.0.3.0. sysCell[2]=0; // ip_begin // begin ip of a high level colon definition
	1.0.3.0. sysCell[3]=0; // boundayCheck=1 for checking
	1.0.3.1. sysCell[4]=0; // data stack depth // dDepth <= DSTKLMT
	1.0.3.3. sysCell[5]=0; // Tib length // nTib < TIBLMT
	1.0.3.2. sysCell[6]=0; // parsing entry // toIn <= nTib
	1.0.3.4. sysCell[7]=0; // execute token // XT // word cfa
	1.0.3.5. sysCell[8]=0; // number of words in dictionary // nWord
	1.0.3.6. sysCell[9]=10; // number conversion base // 2 <= base <= 36
	1.0.4. ramWord_t // { cell_t* prev, cell_t* cfa, char name[] } // cfa points to next cell after name
	1.0.5. ramWord_t ramDict[] // ram wordset
	1.1. rom
	1.1.0. system rom functions
	1.1.0.0. void dStk_top(uint32_t data) {
		if(boundayCheck && (! dDepth)) throw(-2, "no dStk top to put"); }
	1.1.0.1. void dStk_push(uint32_t data) {
		if(boundayCheck && dDepth>=DSTKLMT) throw(-1, "dStk overflow");
		dStk[dDepth++]=data; }
	1.1.0.2. uint32_t dStk_pick(uint8_t i) {
		if(boundayCheck && (i>=dDepth)) throw(-3, "no dStk item-", i, " to pick");
		return dStk[dDepth-1-i]; }
	1.1.0.3. uint32_t dStk_pop(uint8_t i) {
		if(boundayCheck && (i>=dDepth)) throw(-4, "no dStk item-", i, " to pop");
		uint32_t item=dStk[i];
		iTop=dDepth--; for(uint8_t j=iTop-i; j<iTop; j++) dStk[j]=dStk[j+1];
		return item; }
	1.1.0.4. void dStk_roll(uint8_t i) {
		if(boundayCheck && (i>=dDepth)) throw(-5, "no dStk item-", i, " to roll");
		if(! i) return; 
		uint32_t top = dStk[i], iTop=dDepth-1;
		for(uint8_t j=i; j<iTop; j--) dStk[j]=dStk[j+1]; dStk[iTop]=top; }
	1.1.0.6. void dStk_backroll(uint8_t i) {
		if(boundayCheck && (i>=dDepth)) throw(-7, "no dStk item-", i, " to backroll");
		if(! i) return; 
		iTop=dDepth-1; uint32_t top = dStk[iTop];
		for(uint8_t j=iTop; j>i; j--) dStk[j]=dStk[j-1]; dStk[i]=top; }
	1.1.1. forth rom functions
	1.1.1.0. void _call__() { w=*ip++; rStk_push(ip); ip=w; } // complie-only
	1.1.1.0. void _exit__() { ip=rStk_pop(); } // complie-only
	1.1.1.0. void _con__1() { dStk_push(*ip); _exit__(); } // complie-only
	1.1.1.0. void _var__1() { dStk_push(ip); _exit__(); } // complie-only
	1.1.1.0. void _lit__1() { dStk_push(*ip++); } // complie-only
	1.1.1.0. void _dup_12() { dStk_push(dStk_pick(0)); }
	1.1.1.1. void _over_23() { dStk_push(dStk_pick(1)); }
	1.1.1.2. void _drop_10() { dStk_pop(0); }
	1.1.1.3. void _nip_21() { dStk_pop(1); }
	1.1.1.4. void _swap_22() { dStk_roll(1); }
	1.1.1.5. void _rot_33() { dStk_roll(2); }
	1.1.1.6. void _dash_rot_33() { dStk_backroll(2); }
	1.1.1.7. void _pick_nn() { dStk_push(dStk_pick(dStk_pop(0))); }
	1.1.1.8. void _roll_nm() { dStk_roll(dStk_pop(0)); }
	1.1.1.9. void _add_21() { dStk_top(dStk_pick(1)+dStk_pop(0)); }
	1.1.1.10. void _sub_21() { dStk_top(dStk_pick(1)-dStk_pop(0)); }
	1.1.1.11. void _mul_21() { dStk_top(dStk_pick(1)*dStk_pop(0)); }
	1.1.1.12. void _div_21() { dStk_top(dStk_pick(1)/dStk_pop(0)); }
	1.1.1.13. void _and_21() { dStk_Top(dStk_pick(1)&dStk_pop(0)); }
	1.1.1.14. void _or_21() { dStk_Top(dStk_pick(1)|dStk_pop(0)); }
	1.1.1.15. void _xor_21() { dStk_Top(dStk_pick(1)^dStk_pop(0)); }
	1.1.1.16. void _invert_11() { dStk_Top(~dStk_pick(0)); }
	1.1.1.17. void _negate_11() { dStk_Top(-dStk_pick(0)); }
	1.1.1.18. void _eq_21() { dStk_Top(dStk_pick(1)==dStk_pop(0)); }
	1.1.1.19. void _lt_21() { dStk_Top(dStk_pick(1)<dStk_pop(0)); }
	1.1.1.20. void _le_21() { dStk_Top(dStk_pick(1)<=dStk_pop(0)); }
	1.1.1.21. void _eq0_11() { dStk_Top(dStk_pick(0)==0); }
	1.1.1.22. void _lt0_11() { dStk_Top(dStk_pick(0)<0); }
	1.1.1.23. void _le0_11() { dStk_Top(dStk_pick(0)<=0); }
	1.1.1.24. void _not_11() { dStk_Top(!dStk_pick(0)); }
	1.1.1.25. void _dot_10() { serial.print(dStk_pop(0),sysVar[base]); }
	1.1.1.26. void _print_10() { serial.print((char*)dStk_pop(0)); }
	1.1.1.27. void _type_20() { uint8_t n=dStk_pop(0); char*p=dStk_pop(0);
		while(n--)serial.print(*p++); }
	1.1.1.28. void _base_01() { dStk_push(base*sizeof(cell_t)); }
	1.1.1.29. void _fetch_11() { dStk_Top(sysCell[dStk_pick(0)/sizeof(cell_t)]); }
	1.1.1.30. void _c_fetch_11() { dStk_Top(*(((uint8_t*)&sysVar[0])+dStk_pick(0))); }
	1.1.1.31. void _c_store_20() { cell_t p=dStk_pop(0);
		sysByte[p]=(uint8_t)dStk_pop(0); }
	1.1.1.32. void _store_20() { cell_t p=dStk_pop(0)/sizeof(cell_t);
		sysCell[p]=dStk_pop(0); }
	1.1.2. romWord_t // { char* name, func code } // { "\xc0" "if", _if } // IMMRDIATE + COMP_ONLY
	1.1.3. romWord_t romDict[] // rom wordset
