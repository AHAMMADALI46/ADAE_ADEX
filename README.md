# ADAE_ADEX

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

/*ADCM*/


