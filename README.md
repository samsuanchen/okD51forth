# okD51forth
try to build a cpp forth for adafruit samd51 itsybitsy m4
## to do
  0. flashRom size of at most 6 K bytes
  1. 1st project -- forth script interpreter
  1.0. ram (DSTKLMT 32, TIBLMT 128, TSBLMT 128)
  1.0.0. uint32_t dStk[DSTKLMT]; // data stack
  1.0.1. char tib[TIBLMT]; // terminal input buffer
  1.0.2. char tsb[TSBLMT]; // token string buffer
  1.0.3. system variables // cell_t sysCell[32]; uint8_t* sysByte = (uint8_t*) sysCell;
  1.0.3.0. sysCell[0]=0; // boundayCheck
  1.0.3.1. sysCell[1]=0; // data stack depth // dDepth <= DSTKLMT
  1.0.3.3. sysCell[2]=0; // length of Tib // nTib < TIBLMT
  1.0.3.2. sysCell[3]=0; // parsing entry // toIn <= nTib
  1.0.3.4. sysCell[4]=0; // execute token // XT // word cfa
  1.0.3.5. sysCell[5]=0; // number of words in dictionary // nWord
  1.0.3.6. sysCell[6]=10; // number conversion base // 2 <= base <= 36
  1.1. rom
  1.1.0. system functions
  1.1.0.0. void dStk_top(uint32_t data) { 
    if(boundayCheck && (! dDepth)) throw(-2, "no dStk top to put");
  1.1.0.1. void dStk_push(uint32_t data) {
    if(boundayCheck && dDepth>=DSTKLMT) throw(-1, "dStk overflow");
    dStk[dDepth++]=data; }
    dStk[dDepth-1]=data; }
  1.1.0.2. uint32_t dStk_pick(uint8_t i) {
    if(boundayCheck && (i>=dDepth)) throw(-3, "no dStk item-", i, " to pick");
    return dStk[dDepth-1-i]; }
  1.1.0.3. uint32_t dStk_pop(uint8_t i) {
    if(boundayCheck && (i>=dDepth)) throw(-4, "no dStk item-", i, " to pop");
    uint32_t item=dStk[i]; iTop=dDepth--; for(uint8_t j=iTop-i; j<iTop; j++) dStk[j]=dStk[j+1]; return item; }
  1.1.0.4. void dStk_roll(uint8_t i) {
    if(boundayCheck && (i>=dDepth)) throw(-5, "no dStk item-", i, " to roll");
    if(! i) return; 
    uint32_t top = dStk[i], iTop=dDepth-1; for(uint8_t j=i; j<iTop; j--) dStk[j]=dStk[j+1]; dStk[iTop]=top; }
  1.1.0.6. void dStk_backroll(uint8_t i) {
    if(boundayCheck && (i>=dDepth)) throw(-7, "no dStk item-", i, " to backroll");
    if(! i) return; 
    iTop=dDepth-1; uint32_t top = dStk[iTop], for(uint8_t j=iTop; j>i; j--) dStk[j]=dStk[j-1]; dStk[i]=top; }
  1.1.1. primitive functions for each word
  1.1.1.0. void _dup() { dStk_push(dStk_pick(0)); }
  1.1.1.1. void _over() { dStk_push(dStk_pick(1)); }
  1.1.1.2. void _drop() { dStk_pop(0); }
  1.1.1.3. void _nip() { dStk_pop(1); }
  1.1.1.4. void _swap() { dStk_roll(1); }
  1.1.1.5. void _rot() { dStk_roll(2); }
  1.1.1.6. void _dash_rot() { dStk_backroll(2); }
  1.1.1.7. void _pick() { dStk_push(dStk_pick(dStk_pop(0))); }
  1.1.1.8. void _roll() { dStk_roll(dStk_pop(0)); }
  1.1.1.9. void _add() { dStk_top(dStk_pick(1)+dStk_pop(0)); }
  1.1.1.10. void _sub() { dStk_top(dStk_pick(1)-dStk_pop(0)); }
  1.1.1.11. void _mul() { dStk_top(dStk_pick(1)*dStk_pop(0)); }
  1.1.1.12. void _div() { dStk_top(dStk_pick(1)/dStk_pop(0)); }
  1.1.1.13. void _and() { dStk_Top(dStk_pick(1)&dStk_pop(0)); }
  1.1.1.14. void _or() { dStk_Top(dStk_pick(1)|dStk_pop(0)); }
  1.1.1.15. void _xor() { dStk_Top(dStk_pick(1)^dStk_pop(0)); }
  1.1.1.16. void _invert() { dStk_Top(~dStk_pick(0)); }
  1.1.1.17. void _negate() { dStk_Top(-dStk_pick(0)); }
  1.1.1.18. void _eq() { dStk_Top(dStk_pick(1)==dStk_pop(0)); }
  1.1.1.19. void _lt() { dStk_Top(dStk_pick(1)<dStk_pop(0)); }
  1.1.1.20. void _le() { dStk_Top(dStk_pick(1)<=dStk_pop(0)); }
  1.1.1.21. void _eq0() { dStk_Top(dStk_pick(0)==0); }
  1.1.1.22. void _lt0() { dStk_Top(dStk_pick(0)<0); }
  1.1.1.23. void _le0() { dStk_Top(dStk_pick(0)<=0); }
  1.1.1.24. void _not() { dStk_Top(!dStk_pick(0)); }
  1.1.1.24. void _dot() { serial.print(dStk_pop(0),sysVar[base]); }
  1.1.1.24. void _print() { serial.print((char*)dStk_pop(0)); }
  1.1.1.24. void _type() { uint8_t n=dStk_pop(0); char*p=dStk_pop(0); while(n--)serial.print(*p++); }
  1.1.1.24. void _base() { dStk_push(base*sizeof(cell_t)); }
  1.1.1.24. void _fetch() { dStk_Top(sysCell[dStk_pick(0)/sizeof(cell_t)]); }
  1.1.1.24. void _c_fetch() { dStk_Top(*(((uint8_t*)&sysVar[0])+dStk_pick(0))); }
  1.1.1.24. void _c_store() { cell_t p=dStk_pop(0); sysByte[p]=(uint8_t)dStk_pop(0); }
  1.1.1.24. void _store() { cell_t p=dStk_pop(0)/sizeof(cell_t); sysCell[p]=dStk_pop(0); }


1.0.3. Word structure -- { char*name, func code } // { "\xc0" "if", _if } // IMMRDIATE + COMP_ONLY
1.0.4. Word* dict[]={
1.1.2. dictionary of primitive wordset
