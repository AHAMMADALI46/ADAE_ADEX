# ADAE_ADCM

/*ADAE-------Adam Adverse event
structure: one record/
source: adsl, sdtm.ae, sdtm.suppae sdtm.dm*/

options validvarname=upcase nofmterr missing=' ' msglevel=i;

proc sort data=sdtm.ae out=AE; by usubjid; run;
proc sort data=sdtm.dm out=dm; by usubjid; run;
proc sort data=sdtm.suppae out=suppae; by usubjid; run;

Data ae1 (keep=studyid domain usubjid aeseq aespid aeterm aedecod
aeendtc aendt aendtf aestdy aeendy aeenrf aesev aeser aeacn aerel
aesdisab aesdth aeshosp aeslife aesmie aecontrat);
set ae (drop=domain);
domain="ADAE";
Astdt=input(aesdtc, yymmdd10.);
astdtf=" ";
aendt=input(aeendtc, yymmdd10.);
aendtf=" ";
if Aerel in ("Possibly related") then RELGER="Related";
else if AE.AEREL in ("Not related", "unlikely related" then relgr="Not related";
format astdt aendt date9.;
if nmiss(Aendt, astdt)=0 then aedur= (Aendt-aestdt)+1;
run;

Data dm1(keep=usubjid rfstdtc rfxendtc);
set dm;
run;

Proc sort data=ae1; by usubjid; run;
proc sort data=dm1; by usubjid; run;

Data Ae_dm;
merge ae1(in=A) Dm1;
if a;
by usubjid;
run;

Data Ae2(drop=rfstdtc rfxendtc rfstdtn rfxendtn);
set ae_dm;
if rfstdtc ne ' ' then 
rfstdtn=input(rfstdtc, yymmdd10.);
if rfxendtc ne ' ' then 
rfxendtn=input(rfxendtc, yymmdd10.);
astdy=(astdt-rfstdtn+(astdt>=rfstdtn));
aendy=(Aendt-rfstdtc+(aendt>=rfstdtn));
(***************/*Astdy=Analysis start day and Aendy=Analysis end day*/*******)
if rfstdtn<=Astdt<=(rfxendtn+30) then trtemfl="Y";
if astdt<Rfstdtn or rfstdtn=. then prefl="Y";
run;


Data suppae1 (keep=usubjid qnam qval idvarval);
set suppae;
where qnam in ('AECMSPID' 'AEWTHDRW') and QVAL NE " ";
run;

Proc transpose data=suppae1 out=suppae2;
by usubjid idvarval;
id qnam;
var qval;
run;

Proc sort data=suppae2; by usubjid; run;
proc sort data=Ae2; by usubjid; run;

Data adae (drop=_name_ _label_);
merge Ae2 (in=A) suppae2;
by usubjid;
if a;
run;

data Adam.Adae;
merge Adam.adsl (in=A) ADAE;
by usubjid;
if a;
run;

/*ADCM*---------------adam concomitant Medications
Structure: One record/subject/CM/dose-interval
source: Adam.adsl, sdtm.cm*/

options validvarname=upcase nofmterr missing=' ' msglevel=i;
proc sort data=sdtm.cm out=cm; by usubjid; run;

data x;
set cm;

run;

data cm1 (keep=studyid domain usubjid cmseq cmtrt cmindc cmdecod cmspid cmcat cmscat cmdose cm
cmroute epoch cmstdtc_ astdtm astdt asttm astdtf asttmf cmendtc aendtm cmdur cmongo cmongfl);
set cm(drop=Domain);
domain="ADCM";

if cmstdtc ne ' ' then cmstdtc_=cat(cmstdtc // "T"// "00:00:00");
astdt=input(substr(cmstdtc_,1,10),yymmdd10.);
asttm=input(substr(cmstdtc_,12), time8.);
astdtm=input(cmstdtc_, anydtdtm.);
format astdt date9. asttm time8. astdtm datetime20.;

if cmendtc ne ' ' then cmendtc_=cat(cmendtc // "T"// "00:00:00");
aendt=input(substr(cmendtc_,1,10),yymmdd10.);
aentm=input(substr(cmendtc_,12), time8.);
aendtm=input(cmstdtc_, anydtdtm.);
format aendt date9. aentm time8. aendtm datetime20.;
Astdtf=" " ;
asttmf="Y";
Aendtf=" ";
Aendtmf="Y";

if Nmiss (Aendt, astdt)=0 then cmdur=(aendt-astdt)+1;
cmongo=cmenrf;
cmongfl="Y"; else cmongfl=" "
run;

Proc sort data=sdtm.suppcm out=suppcm; by usubjid; run;

data suppcm1 (keep=usubjid qnam qval, idvarval);
set suppcm;
where qnam in ("Aenum" "MhNum" "OTHINDC") and Qval ne " ";
run;

Proc sort data=suppcm1; by usubjid idvarval; run;

Proc transpose data=suppcm1 out=suppcm2;
by usubjid idvarval;
id qnam;
var qval;
run;

proc sort data=suppcm2; by usubjid; run;
proc sort data=cm1; by usubjid; run;

data adcm ;
merge cm1 (rename=(cmstdtc_=cmstdtc cmendtc_=cmendtc)) suppcm2 (drop=_name_ _label_ idvarval) ;
by usubjid;
run;

data adam.adcm;
merfe adam.adsl adcm (in=A);
by usubjid;
if a;
run;


/*ADEX---------ADAM EXPOSURE*/
Source: Adam.adsl, sdtm.ex, sdtm.suppex*/

options validvarname=upcase nofmterr missing=' ' msglevel=i;

Proc sort data=sdtm.ex out=ex; by usubjid; run;

Data x;
set ex;
if exstdtc ne ' ' and length (exstdtc) ge 16 then exstdtc_=Exstdtc;
else if exstdtc ne ' ' and length(exstdtc) ne 16 then exstdtc=Cat(exstdtc//"T"//"00.00");
run;

data ex1 (keep=studyid domain usubjid exseq exgrpid extrt excat exscat
epoch exstdtc exendtc exstdy exendy astdtm astdt asttm
aentmf);
set ex (drop=domain);
domain="ADEX";
exstdt=input(substr(exstdtc, 1, 10), YYMMDD10.);
if length(exstdtc) ge 16 then 
exstm=input(substr(exstdtc, 12), time8.);
else if length (exstdtc) lt 16 then exstm=input("00:00", time8
if exstdtc ne' ' and length(exstdtc) ge 16 then exstdtc_=exstdtc;
else if exstdtc Ne ' ' and length(exstdtc) Ne 16 then exstdtc_=CAT(Exstdtc //"T"//"00.00");
ASTDTM=input (exstdtc, yymmdd10.);
ASTDT=input(exstdtc, yymmdd10.);
asttm=input(exstdtc, yymmdd10.);
astdtf=" ";
asttmf=" ";







































































