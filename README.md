# PLC Functional Block Code Generator

## Overview  
This project provides an automated way to **generate PLC functional block code** using **Google Sheets**. The script utilizes a **custom formula** in Google Sheets (LET function) to parse structured tabular data and output **ready-to-use PLC function blocks**.

The generated code is formatted in **Structured Text (ST)** for Siemens PLCs and follows an optimized pattern for **functional block automation**.

---

## Features  
-Automated Code Generation – Converts structured tabular data into PLC function blocks  
-Supports Multiple Devices – Generates commands for **valves, pumps, sensors, and motors**  
-Dynamic Data Parsing – Extracts relevant functional elements and maps them into PLC syntax  
-Works in Google Sheets – Uses `LET`, `TEXTJOIN`, and `ARRAYFORMULA` functions  
---
##  How It Works  
1. Input Data in a Table – Define actions, device names, and comments in structured columns  
2. Formula Processes Data – Parses each row and generates function block code  
3. Outputs Structured Text Code – Generates PLC function blocks dynamically  
---
##  PLC Functional Block Table Example  

| No. | Ref No. | Phase | Step | Description | CIP/Water Supply | Transfer/Return | General Devices |
|----|--------|------|----|------------|------------------|-----------------|----------------|
| 1  | 001    | 1    | 1  | Initial Setup | V001 - Turn OFF (Close main CIP valve) <br> V002 - Turn OFF (Close drain valve) <br> V003 - Turn OFF (Stop water supply) <br> V004 - Turn OFF (Close bypass valve) | P001 - Stop (Transfer pump) <br> V101 - Turn OFF (Close return valve) <br> V102 - Turn OFF (Disable syrup flow) <br> V103 - Turn OFF (Close supply line) | M001 - Turn OFF (Rotary valve motor) <br> A001 - Turn OFF (Air supply to dosing system) <br> V201 - Turn OFF (Shut off product recirculation) <br> V202 - Turn OFF (Close main tank drain) |
| 2  | 002    | 1    | 2  | Pre-Rinse Process | V005 - Turn ON (Activate rinse water flow) <br> V006 - Turn ON (Open drain valve) <br> V007 - Turn ON (Enable pre-rinse bypass) <br> S001 - Check (Water pressure sensor) | P002 - Start (Pre-rinse pump) <br> V104 - Turn ON (Open return line) <br> V105 - Turn ON (Enable return flow)| M002 - Turn ON (Mixing motor) <br> A002 - Turn ON (Activate air injection) <br> V203 - Turn ON (Recirculation loop) <br> X001 - Start (Mixing process) |
| 3  | 003    | 2    | 1  | Main Washing Cycle | V008 - Turn ON (Open detergent valve) <br> V009 - Turn ON (Enable main wash cycle) <br> V010 - Turn ON (Activate heat exchanger) <br> S003 - Check (Temperature sensor) | P003 - Start (Main wash pump) <br> V106 - Turn ON (Enable detergent return) <br> V107 - Turn ON (Open secondary flow) | M003 - Turn ON (Heater activation) <br> A003 - Turn ON (Open air purge) <br> V204 - Turn ON (Drain wash system) <br> X002 - Start (Circulation process) |
| 4  | 004    | 2    | 2  | Rinse & Drain | V011 - Turn OFF (Close detergent valve) <br> V012 - Turn ON (Enable rinse water) <br> V013 - Turn ON (Open final drain) <br> S005 - Check (Flow sensor) | P004 - Stop (Main wash pump) <br> V108 - Turn OFF (Close return line) <br> V109 - Turn ON (Final rinse activation)| M004 - Turn OFF (Cooling system) <br> A004 - Turn OFF (Deactivate purge air) <br> V205 - Turn OFF (Close recirculation) <br> X003 - Stop (Final processing step) |

---

### **Google Sheets Formula**
```
=LET(
  fullText; TEXTJOIN(CHAR(10); TRUE; F2:H2);
  fCol; ARRAYFORMULA(TRIM(INDEX(SPLIT(TRANSPOSE(SPLIT(fullText; CHAR(10))); "-"); 0; 1)));
  sCol; ARRAYFORMULA(TRIM(IFERROR(INDEX(SPLIT(TRANSPOSE(SPLIT(fullText; CHAR(10))); "-"); 0; 2); "")));
  validRows; ARRAYFORMULA(sCol <> "");
  header; "FUNCTION " & CHAR(34) & "P" & C2 & "S" & D2 & CHAR(34) & " : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_IN_OUT 
      My :" & CHAR(34) & "auto_SDT" & CHAR(34) & ";
   END_VAR
BEGIN" & CHAR(10);
  comment; ARRAYFORMULA(
    IFERROR(
      MID(
        sCol;
        SEARCH("("; sCol) + 1;
        SEARCH(")"; sCol) - SEARCH("("; sCol) - 1
      );
      ""
    )
  );
  result; ARRAYFORMULA(
    IF(validRows;
       CHAR(34) & fCol & CHAR(34) & "." & 
       IF(REGEXMATCH(fCol; "(?i)v"); "cmdOpen"; "cmdStart") & 
       IF(REGEXMATCH(sCol; "Turn OFF"); ":=FALSE"; ":=TRUE") & "; //" & comment;
       "//" & fCol
    )
  );
  IF(fullText = ""; ""; header & ARRAYFORMULA(IFERROR(JOIN(CHAR(10); result); "")) &  CHAR(10) & "END_FUNCTION")
)
```
---
### **Output formula example**
```
FUNCTION "P1S1" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_IN_OUT 
My :"auto_SDT";
END_VAR
BEGIN
"V001".cmdOpen:=FALSE; //Close main CIP valve
"V002".cmdOpen:=FALSE; //Close drain valve
"V003".cmdOpen:=FALSE; //Stop water supply
"V004".cmdOpen:=FALSE; //Close bypass valve
"P001".cmdStart:=TRUE; //Transfer pump
"V101".cmdOpen:=FALSE; //Close return valve
"V102".cmdOpen:=FALSE; //Disable syrup flow
"V103".cmdOpen:=FALSE; //Close supply line
"M001".cmdStart:=FALSE; //Rotary valve motor
"A001".cmdStart:=FALSE; //Air supply to dosing system
"V201".cmdOpen:=FALSE; //Shut off product recirculation
"V202".cmdOpen:=FALSE; //Close main tank drain
END_FUNCTION

```
---

## Contributors
- [Evgenii-Zinner](https://github.com/Evgenii-Zinner) - Co-creator
- [Azamat Choorov](https://github.com/dokoni-mo) - Co-creator
