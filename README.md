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
if Aerel in ("
