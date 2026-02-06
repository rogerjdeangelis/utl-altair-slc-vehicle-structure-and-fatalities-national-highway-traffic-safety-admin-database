# utl-altair-slc-vehicle-structure-and-fatalities-national-highway-traffic-safety-admin-database
Vehicle structure and fatalities from national highway traffic safety administration database
    %let pgm=utl-altair-slc-vehicle-structure-and-fatalities-national-highway-traffic-safety-admin-database;

    %stop_submission;

    Vehicle structure and fatalities from national highway traffic safety administration database

    Too long to post here,see github
    https://github.com/rogerjdeangelis/utl-altair-slc-vehicle-structure-and-fatalities-national-highway-traffic-safety-admin-database

    Problem
        Indentify Vehicle Structual and Crash Related Information invoved in Vehicle Fatalities

    CONTENTS

        1 Process (how to add fatalities to Vehicle Structual and Crash Related Information )
        2 Inputs
        3 All tables and columns

    DOWNLOAD

    https://www.nhtsa.gov/file-downloads?p=nhtsa/downloads/FARS/2023/National/
    FARS2023NationalSAS.zip  (unzip into e:/fars)

    META DATA

    Table (inn e:/fars)  Keys

    vpicdecode.sas7bdat  vehicledescriptor with missing 9th letter(check digit)  (203 variables x 57,145 obs many structual variables?)
    accident.sas7bdat    st_case and fatals (49 variables x 37,654 obs. st_case is a primary key)
    vehicle.sas7bdat     st_case and vin (110 x 58,319 variables)


    STRUCTUAL AND CRASH RELATED INFORMATION AVAILABLE (ALMOST 60,000 VINS?)

        TABLE         COLUMN                STRUCTURE


       VSOE           AOI                   Areas of Impact
       CEVENT         AOI2                  Areas of Impact (Other Vehicle)
       CEVENT         AOI1                  Areas of Impact (This Vehicle)
       DAMAGE         DAMAGE                Areas of Impact - Damaged Areas
       PARKWORK       PIMPACT1              Areas of Impact - Initial Contact Point
       VPICDECODE     TRUCKBEDLENGTHIN      Bed Length (inches)
       VPICDECODE     TRUCKBEDTYPE          Bed Type
       VPICDECODE     TRUCKBEDTYPEID        Bed Type ID
       VPICDECODE     BODYCLASS             Body Class
       VPICDECODE     TRUCKBODYCABTYPE      Cab Type
       PARKWORK       PCARGTYP              Cargo Body Type
       VEHICLE        ACC_CONFIG            Crash Type Configuration
       VPICDECODE     DRIVETYPE             Drive Type
       VPICDECODE     ENGINECONFIGURATION   Engine Configuration
       VPICDECODE     ENGINECYLINDERSCOUNT  Engine Number of Cylinders
       VEHICLE        J_KNIFE               Jackknife
       VEHICLE        IMPACT1               Areas of Impact - Initial Contact Point
       ACCIDENT       MAN_COLL              Manner of Collision of the First Harmful Event
       ACCIDENT       PERSONS               Number of MV Occupant
       VPICDECODE     SEATROWSCOUNT         Number of Seat Rows
       VPICDECODE     SEATSCOUNT            Number of Seats
       VPICDECODE     WHEELSCOUNT           Number of Wheels
       PBTYPE         PEDCGP                Pedestrian Crash Group
       PBTYPE         PEDLOC                Pedestrian Crash Location
       PBTYPE         PEDCTYPE              Pedestrian Crash Type
       VEHICLE        PCRASH5               Pre- Impact Location
       VEHICLE        PCRASH4               Pre- Impact Stability
       PERSON         ROLLOVER              Rollover
       VPICDECODE     SERIES                Series (ie 2-dr Convertible Body Style)
       VPICDECODE     SERIES2               Series2
       VPICDECODE     STEERINGLOCATION      Steering Location
       VPICDECODE     ENGINETURBOID         Turbo ID
       PERSON         REST_USE              Type of Restraint System in Use
       VPICDECODE     WHEELBASEIN_FROM      Wheel Base (inches) From
       VPICDECODE     WHEELBASEIN_TO        Wheel Base (inches) To
       VPICDECODE     WHEELBASETYPE         Wheel Base Type
       VPICDECODE     WINDOWS               Windows
       PARKWORK       PVPICBODYCLASS        vPIC Body Class

    /*
    / | _ __  _ __ ___   ___ ___  ___ ___
    | || `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | || |_) | | | (_) | (_|  __/\__ \__ \
    |_|| .__/|_|  \___/ \___\___||___/___/
       |_|
    */

    proc datasets lib=workx kill;
    run;quit;

    libname efars sas7bdat "e:/fars";

    proc sql;
      create
         table workx.acdCase as
      select
         l.st_case
        ,l.fatals
        ,r.vin
        ,cats(substr(vin,1,8),'*',substr(vin,10,2)) as vinles
      from
         efars.accident as l left join efars.vehicle as r
      on
         l.st_case = r.st_case
    ;quit;

    /*--- Remove duplicates - suspect vehicles are the same but with slighyly different vins or vin is wrong? ---*/

    proc sort data=workx.acdCase out=workx.acdCaseUnq noequals nodupkey;
     by st_case fatals;
    ;quit;

    /*--- NOTE: 20665 observations were found and deleted due to having duplicate sort keys ---*/


    /**************************************************************************************************************************/
    /*  WORKX.ACDCASEUNQ total obs=37,654                                                                                     */
    /*                                                                                                                        */
    /*   Obs    ST_CASE    FATALS        VIN           VINLES                                                                 */
    /*                                                                                                                        */
    /*     1      10001       1      1FMDU32X8TUD    1FMDU32X*TU                                                              */
    /*     2      10002       1      1B3CB3HAXAD6    1B3CB3HA*AD                                                              */
    /*     3      10003       1      4T1BE32K16U7    4T1BE32K*6U                                                              */
    /*     4      10004       1      3C4NJDBB9JT1    3C4NJDBB*JT                                                              */
    /*     5      10005       1      4T1BF12B3TU1    4T1BF12B*TU                                                              */
    /*     ....                                                                                                               */
    /*                                                                                                                        */
    /* 37650     560117       1      1C6RR7FT5FS6    1C6RR7FT*FS                                                              */
    /* 37651     560118       1      1GNDT13W02K1    1GNDT13W*2K                                                              */
    /* 37652     560119       1      1FUJHLFG0LLL    1FUJHLFG*LL                                                              */
    /* 37653     560120       1      5YJYGDEE5LF0    5YJYGDEE*LF                                                              */
    /* 37654     560121       1      4V4NC9EJ1EN1    4V4NC9EJ*EN                                                              */
    /**************************************************************************************************************************/

    libname efars sas7bdat "e:/fars";

    proc sql;
      create
         table workx.addstruct as
      select
          l.st_case
         ,l.fatals
         ,l.vin as fulvin
         ,r.make
         ,r.drivetype
         ,r.series2
         ,r.bodyclass
         ,r.engineconfiguration
         ,r.seatbelttype
         ,r.doorscount
         ,r.wheelscount
         ,r.displacementl
      from
         workx.acdCaseUnq as l, efars.vpicdecode as r
      where
         strip(l.vinles) = substr(r.vehicledescriptor,1,11)
      order
         by st_case
    ;quit;


    proc sort data=workx.addstruct out=workx.addstructUnq nodupkey;
     by st_case descending fatals;
    run;quit;

    /*--- NOTE: 168063 observations were found and deleted due to having duplicate sort keys ---*/

    /**************************************************************************************************************************/
    /*  WORKX.ADDSTRUCTUNQ total obs=35,702                                                                                   */
    /*                                                                                                                        */
    /* Obs ST_CASE FATALS  FULVIN    MAKE   DRIVETYPE         SERIES2 BODYCLASS                                               */
    /*                                                                                                                        */
    /*   1  10001     1 1FMDU32X8TUD FORD                             Sport Utility Vehicle (SUV)/Multi-Purpose Vehicle (MPV) */
    /*   2  10002     1 1B3CB3HAXAD6 DODGE  FWD/Front-Wheel Drive     Hatchback/Liftback/Notchback                            */
    /*   3  10003     1 4T1BE32K16U7 TOYOTA 4x2                       Sedan/Saloon                                            */
    /*   4  10004     1 3C4NJDBB9JT1 JEEP   4WD/4-Wheel Drive/4x4     Sport Utility Vehicle (SUV)/Multi-Purpose Vehicle (MPV) */
    /*   5  10005     1 4T1BF12B3TU1 TOYOTA 4x2                       Sedan/Saloon                                            */
    /*                                                                                                                        */
    /* Obs ENGINECONFIGURATION SEATBELTTYPE DOORSCOUNT WHEELSCOUNT DISPLACEMENTL                                              */
    /*                                                                                                                        */
    /*   1      V-Shaped          Manual         4          .            4                                                    */
    /*   2                        Manual         4          .            2                                                    */
    /*   3      In-Line           Manual         4          .            2                                                    */
    /*   4      In-Line           Manual         4          4            2                                                    */
    /*   5      V-Shaped          Manual         4          .            3                                                    */
    /**************************************************************************************************************************/



     ____   _                   _
    |___ \ (_)_ __  _ __  _   _| |_ ___
      __) || | `_ \| `_ \| | | | __/ __|
     / __/ | | | | | |_) | |_| | |_\__ \
    |_____||_|_| |_| .__/ \__,_|\__|___/
                   |_|     _                    _        _        _     _
    __   ___ __ (_) ___ __| | ___  ___ ___   __| | ___  | |_ __ _| |__ | | ___
    \ \ / / `_ \| |/ __/ _` |/ _ \/ __/ _ \ / _` |/ _ \ | __/ _` | `_ \| |/ _ \
     \ V /| |_) | | (_| (_| |  __/ (_| (_) | (_| |  __/ | || (_| | |_) | |  __/
      \_/ | .__/|_|\___\__,_|\___|\___\___/ \__,_|\___|  \__\__,_|_.__/|_|\___|
          |_|
    */

    ENGINECONFIGURATION

    Middle Observation(28572 ) of table = efars.vpicdecode


     -- CHARACTER --
    Variable                        Typ    Value                         Label

    VEHICLEDESCRIPTOR                C17   JTEBU17R*40******             Masked VIN
    VINDECODEDON                     C20   2024-08-22 00:30:01           VIN Decoded On
    VINDECODEERROR                   C50   0                             Error Code
    VEHICLETYPE                      C40   MULTIPURPOSE PASSENGER VEHI   Vehicle Type
    MANUFACTURERFULLNAME             C135  TOYOTA MOTOR NORTH AMERICA,   Manufacturer Name
    MAKE                             C80   TOYOTA                        Make
    MODEL                            C140  4-Runner                      Model
    SERIES                           C165  GRN210L/GRN215L/UZN210L/UZN   Series
    TRIM                             C160  Limited                       Trim
    SERIES2                          C65   Wagon body style              Series2
    TRIM2                            C160                                Trim2
    PLANTCOUNTRY                     C40   JAPAN                         Plant Country
    PLANTSTATE                       C30   AICHI                         Plant State
    PLANTCITY                        C45   TAHARA                        Plant City
    PLANTCOMPANYNAME                 C55   Toyota Motor Corporation      Plant Company Name
    DESTINATIONMARKET                C50                                 Destination Market
    NONLANDUSE                       C20                                 Non-Land Use
    NOTE                             C500                                Note
    BODYCLASS                        C80   Sport Utility Vehicle (SUV)   Body Class
    WHEELBASETYPE                    C15                                 Wheel Base Type
    GROSSVEHICLEWEIGHTRATINGFROM     C55   Class 1D: 5,001 - 6,000 lb    Gross Vehicle Weight Rating From
    GROSSVEHICLEWEIGHTRATINGTO       C55                                 Gross Vehicle Weight Rating To
    GROSSCOMBWEIGHTRATINGFROM        C55                                 Gross Combination Weight Rating From
    GROSSCOMBWEIGHTRATINGTO          C55                                 Gross Combination Weight Rating To
    TRUCKBODYCABTYPE                 C45                                 Cab Type
    TRUCKBEDTYPE                     C15                                 Bed Type
    BUSTYPE                          C20   Not Applicable                Bus Type
    BUSFLOORCONFIGURATIONTYPE        C20   Not Applicable                Bus Floor Configuration Type
    OTHERBUSINFO                     C500                                Other Bus Info
    CUSTOMMOTORCYCLETYPE             C25   Not Applicable                Custom Motorcycle Type
    MOTORCYCLESUSPENSIONTYPE         C40   Not Applicable                Motorcycle Suspension Type
    MOTORCYCLECHASSISTYPE            C50   Not Applicable                Motorcycle Chassis Type
    OTHERMOTORCYCLEINFO              C500                                Other Motorcycle Info
    STEERINGLOCATION                 C25                                 Steering Location
    ENTERTAINMENTSYSTEM              C30                                 Entertainment System
    TRANSMISSIONSTYLE                C45                                 Transmission Style
    DRIVETYPE                        C25   4WD/4-Wheel Drive/4x4         Drive Type
    AXLECONFIGURATION                C25                                 Axle Configuration
    BRAKESYSTEMTYPE                  C20                                 Brake System Type
    BRAKESYSTEMDESC                  C500                                Brake System Description
    EVDRIVEUNIT                      C15                                 EV Drive Unit
    BATTERYTYPE                      C30                                 Battery Type
    OTHERBATTERYINFO                 C500                                Battery Info
    CHARGERLEVEL                     C135                                Charger Level
    ENGINEMANUFACTURER               C40   Toyota                        Engine Manufacturer
    ENGINEMODEL                      C115  1GR-FE                        Engine Model
    ENGINECONFIGURATION              C35   V-Shaped                      Engine Configuration
    ENGINECOOLINGTYPE                C10   Water                         Cooling Type
    FUELTYPEPRIMARY                  C45   Gasoline                      Fuel Type - Primary
    FUELTYPESECONDARY                C45                                 Fuel Type - Secondary
    FUELDELIVERYINJECTIONTYPE        C50                                 Fuel Delivery / Fuel Injection Type
    ENGINEVALVETRAINDESIGN           C35   Dual Overhead Cam (DOHC)      Valve Train Design
    ENGINEELECTRIFICATIONLEVEL       C45                                 Electrification Level
    ENGINETURBO                      C10                                 Turbo
    OTHERENGINEINFO                  C500                                Other Engine Info
    SEATBELTTYPE                     C25                                 Seat Belt Type
    PRETENSIONER                     C10                                 Pretensioner
    AIRBAGLOCFRONT                   C35                                 Front Air Bag Locations
    AIRBAGLOCKNEE                    C35                                 Knee Air Bag Locations
    AIRBAGLOCSIDE                    C35                                 Side Air Bag Locations
    AIRBAGLOCCURTAIN                 C35                                 Curtain Air Bag Locations
    AIRBAGLOCSEATCUSHION             C35                                 Seat Cushion Air Bag Locations
    OTHERRESTRAINTSYSTEMINFO         C500                                Other Restraint System Info
    FORWARDCOLLISIONWARNING          C20                                 Forward Collision Warning (FCW)
    DYNAMICBRAKESUPPORT              C20                                 Dynamic Brake Support (DBS)
    CRASHIMMINENTBRAKING             C20                                 Crash Imminent Braking (CIB)
    PEDESTRIANAUTOEMERGENCYBRAKING   C20                                 Pedestrian Automatic Emergency Braking (
    BLINDSPOTWARNING                 C20                                 Blind Spot Warning (BSW)
    BLINDSPOTINTERVENTION            C20                                 Blind Spot Intervention (BSI)
    LANEDEPARTUREWARNING             C20                                 Lane Departure Warning (LDW)
    LANEKEEPINGASSISTANCE            C20                                 Lane Keeping Assistance (LKA)
    LANECENTERINGASSISTANCE          C20                                 Lane Centering Assistance
    BACKUPCAMERA                     C20                                 Backup Camera
    REARCROSSTRAFFICALERT            C20                                 Rear Cross Traffic Alert
    REARAUTOMATICEMERGENCYBRAKING    C20                                 Rear Automatic Emergency Braking
    PARKASSIST                       C20                                 Parking Assist
    DAYTIMERUNNINGLIGHT              C20                                 Daytime Running Light (DRL)
    HEADLAMPLIGHTSOURCE              C30                                 Headlamp Light Source
    SEMIAUTOHEADLAMPBEAMSWITCHING    C20                                 Semiautomatic Headlamp Beam Switching
    ADAPTIVEDRIVINGBEAM              C20                                 Adaptive Driving Beam (ADB)
    ADAPTIVECRUISECONTROL            C20                                 Adaptive Cruise Control (ACC)
    ANTILOCKBRAKESYSTEM              C20                                 Anti-lock Braking System (ABS)
    ELECTRONICSTABILITYCONTROL       C20                                 Electronic Stability Control (ESC)
    TPMS                             C15                                 Tire Pressure Monitoring System (TPMS) T
    AUTOMATICCRASHNOTIFICATION       C20                                 Automatic Crash Notification (ACN) / Adv
    EVENTDATARECORDER                C20                                 Event Data Recorder (EDR)
    TRACTIONCONTROL                  C20                                 Traction Control
    AUTOPEDESTRIANALERTINGSOUND      C20                                 Automatic Pedestrian Alerting Sound (for
    KEYLESSIGNITION                  C20                                 Keyless Ignition
    AUTOREVERSESYSTEM                C20                                 Auto-Reverse System for Windows and Sunr
    ACTIVESAFETYSYSNOTE              C500                                Active Safety System Note


     -- NUMERIC --
    STATE                            N8              27                  State Number
    ST_CASE                          N8          270064                  Consecutive Number
    VEH_NO                           N8               1                  Vehicle Number
    VEHICLETYPEID                    N8               7                  Vehicle Type ID
    MANUFACTURERFULLNAMEID           N8             962                  Manufacturer Name ID
    MAKEID                           N8             448                  Make ID
    MODELID                          N8            2216                  Model ID
    MODELYEAR                        N8            2004                  Model Year
    PLANTCOUNTRYID                   N8               3                  Plant Country ID
    DESTINATIONMARKETID              N8               .                  Destination Market ID
    BASEPRICE                        N8               .                  Base Price ($)
    NONLANDUSEID                     N8               .                  Non-Land Use ID
    BODYCLASSID                      N8               7                  Body Class ID
    DOORSCOUNT                       N8               5                  Doors
    WINDOWS                          N8               .                  Windows
    WHEELBASETYPEID                  N8               .                  Wheel Base Type ID
    TRACKWIDTHIN                     N8               .                  Track Width (inches)
    GROSSVEHICLEWEIGHTRATINGFROMID   N8              13                  Gross Vehicle Weight Rating From ID
    GROSSVEHICLEWEIGHTRATINGTOID     N8               .                  Gross Vehicle Weight Rating To ID
    GROSSCOMBWEIGHTRATINGFROMID      N8               .                  Gross Combination Weight Rating From ID
    GROSSCOMBWEIGHTRATINGTOID        N8               .                  Gross Combination Weight Rating To ID
    CURBWEIGHTLB                     N8               .                  Curb Weight (pounds)
    WHEELBASEIN_FROM                 N8               .                  Wheel Base (inches) From
    WHEELBASEIN_TO                   N8               .                  Wheel Base (inches) To
    WHEELSCOUNT                      N8               .                  Number of Wheels
    WHEELSIZEFRONTIN                 N8               .                  Wheel Size Front (inches)
    WHEELSIZEREARIN                  N8               .                  Wheel Size Rear (inches)
    TRUCKBODYCABTYPEID               N8               .                  Cab Type ID
    TRUCKBEDTYPEID                   N8               .                  Bed Type ID
    TRUCKBEDLENGTHIN                 N8               .                  Bed Length (inches)
    BUSTYPEID                        N8               0                  Bus Type ID
    BUSFLOORCONFIGURATIONTYPEID      N8               0                  Bus Floor Configuration Type ID
    BUSLENGTHFT                      N8               .                  Bus Length (feet)
    CUSTOMMOTORCYCLETYPEID           N8               0                  Custom Motorcycle Type ID
    MOTORCYCLESUSPENSIONTYPEID       N8               0                  Motorcycle Suspension Type ID
    MOTORCYCLECHASSISTYPEID          N8               0                  Motorcycle Chassis Type ID
    STEERINGLOCATIONID               N8               .                  Steering Location ID
    ENTERTAINMENTSYSTEMID            N8               .                  Entertainment System ID
    SEATSCOUNT                       N8               .                  Number of Seats
    SEATROWSCOUNT                    N8               .                  Number of Seat Rows
    TRANSMISSIONSPEEDS               N8               .                  Transmission Speeds
    TRANSMISSIONSTYLEID              N8               .                  Transmission Style ID
    DRIVETYPEID                      N8               2                  Drive Type ID
    AXLESCOUNT                       N8               .                  Axles
    AXLECONFIGURATIONID              N8               .                  Axle Configuration ID
    BRAKESYSTEMTYPEID                N8               .                  Brake System Type ID
    EVDRIVEUNITID                    N8               .                  EV Drive Unit ID
    BATTERYKWH_FROM                  N8               .                  Battery Energy (kWh) From
    BATTERYKWH_TO                    N8               .                  Battery Energy (kWh) To
    BATTERYV_FROM                    N8               .                  Battery Voltage (Volts) From
    BATTERYV_TO                      N8               .                  Battery Voltage (Volts) To
    BATTERYA_FROM                    N8               .                  Battery Current (Amps) From
    BATTERYA_TO                      N8               .                  Battery Current (Amps) To
    BATTERYPACKSPERVEHICLE           N8               .                  Number of Battery Packs per Vehicle
    BATTERYMODULESPERPACK            N8               .                  Number of Battery Modules per Pack
    BATTERYCELLSPERMODULE            N8               .                  Number of Battery Cells per Module
    BATTERYTYPEID                    N8               .                  Battery Type ID
    CHARGERLEVELID                   N8               .                  Charger Level ID
    CHARGERPOWERKW                   N8               .                  Charger Power (kW)
    ENGINECONFIGURATIONID            N8               2                  Engine Configuration ID
    ENGINEPOWERKW                    N8               .                  Engine Power (kW)
    ENGINESTROKECYCLES               N8               4                  Engine Stroke Cycles
    ENGINECYLINDERSCOUNT             N8               6                  Engine Number of Cylinders
    ENGINEBRAKEHP_FROM               N8               .                  Engine Brake (hp) From
    ENGINEBRAKEHP_TO                 N8               .                  Engine Brake (hp) To
    ENGINECOOLINGTYPEID              N8               2                  Cooling Type ID
    DISPLACEMENTCI                   N8             244                  Displacement (CI)
    DISPLACEMENTCC                   N8            4000                  Displacement (CC)
    DISPLACEMENTL                    N8               4                  Displacement (L)
    FUELTYPEPRIMARYID                N8               4                  Fuel Type - Primary ID
    FUELTYPESECONDARYID              N8               .                  Fuel Type - Secondary ID
    FUELDELIVERYINJECTIONTYPEID      N8               .                  Fuel Delivery / Fuel Injection Type ID
    ENGINEVALVETRAINDESIGNID         N8               2                  Valve Train Design ID
    ENGINEELECTRIFICATIONLEVELID     N8               .                  Electrification Level ID
    ENGINETURBOID                    N8               .                  Turbo ID
    TOPSPEEDMPH                      N8               .                  Top Speed (MPH)
    SEATBELTTYPEID                   N8               .                  Seat Belt Type ID
    PRETENSIONERID                   N8               .                  Pretensioner ID
    AIRBAGLOCFRONTID                 N8               .                  Front Air Bag Locations ID
    AIRBAGLOCKNEEID                  N8               .                  Knee Air Bag Locations ID
    AIRBAGLOCSIDEID                  N8               .                  Side Air Bag Locations ID
    AIRBAGLOCCURTAINID               N8               .                  Curtain Air Bag Locations ID
    AIRBAGLOCSEATCUSHIONID           N8               .                  Seat Cushion Air Bag Locations ID
    FORWARDCOLLISIONWARNINGID        N8               .                  Forward Collision Warning (FCW) ID
    DYNAMICBRAKESUPPORTID            N8               .                  Dynamic Brake Support (DBS) ID
    CRASHIMMINENTBRAKINGID           N8               .                  Crash Imminent Braking (CIB) ID
    PEDESTRIANAUTOEMERGENCYBRAKINGID N8               .                  Pedestrian Automatic Emergency Braking (
    BLINDSPOTWARNINGID               N8               .                  Blind Spot Warning (BSW) ID
    BLINDSPOTINTERVENTIONID          N8               .                  Blind Spot Intervention (BSI) ID
    LANEDEPARTUREWARNINGID           N8               .                  Lane Departure Warning (LDW) ID
    LANEKEEPINGASSISTANCEID          N8               .                  Lane Keeping Assistance (LKA) ID
    LANECENTERINGASSISTANCEID        N8               .                  Lane Centering Assistance ID
    BACKUPCAMERAID                   N8               .                  Backup Camera ID
    REARCROSSTRAFFICALERTID          N8               .                  Rear Cross Traffic Alert ID
    REARAUTOMATICEMERGENCYBRAKINGID  N8               .                  Rear Automatic Emergency Braking ID
    PARKASSISTID                     N8               .                  Parking Assist ID
    DAYTIMERUNNINGLIGHTID            N8               .                  Daytime Running Light (DRL) ID
    HEADLAMPLIGHTSOURCEID            N8               .                  Headlamp Light Source ID
    SEMIAUTOHEADLAMPBEAMSWITCHINGID  N8               .                  Semiautomatic Headlamp Beam Switching ID
    ADAPTIVEDRIVINGBEAMID            N8               .                  Adaptive Driving Beam (ADB) ID
    ADAPTIVECRUISECONTROLID          N8               .                  Adaptive Cruise Control (ACC) ID
    ANTILOCKBRAKESYSTEMID            N8               .                  Anti-lock Braking System (ABS) ID
    ELECTRONICSTABILITYCONTROLID     N8               .                  Electronic Stability Control (ESC) ID
    TPMSID                           N8               .                  Tire Pressure Monitoring System (TPMS) T
    AUTOMATICCRASHNOTIFICATIONID     N8               .                  Automatic Crash Notification (ACN) / Adv
    EVENTDATARECORDERID              N8               .                  Event Data Recorder (EDR) ID
    TRACTIONCONTROLID                N8               .                  Traction Control ID
    AUTOPEDESTRIANALERTINGSOUNDID    N8               .                  Automatic Pedestrian Alerting Sound (for
    KEYLESSIGNITIONID                N8               .                  Keyless Ignition ID
    SAEAUTOMATIONLEVEL_FROM          N8               .                  SAE Automation Level From
    SAEAUTOMATIONLEVEL_TO            N8               .                  SAE Automation Level To
    AUTOREVERSESYSTEMID              N8               .                  Auto-Reverse System for Windows and Sunr

    /*         _     _      _        _        _     _
    __   _____| |__ (_) ___| | ___  | |_ __ _| |__ | | ___
    \ \ / / _ \ `_ \| |/ __| |/ _ \ | __/ _` | `_ \| |/ _ \
     \ V /  __/ | | | | (__| |  __/ | || (_| | |_) | |  __/
      \_/ \___|_| |_|_|\___|_|\___|  \__\__,_|_.__/|_|\___|

    */


    Middle Observation(29159 ) of table = efars.vehicle


     -- CHARACTER --
    Variable                        Typ    Value                      Label

    VIN                              C12   3GNBABFWXBS5               Vehicle Identification Number
    TRLR1VIN                         C12   777777777777               Trailer VIN (1)
    TRLR2VIN                         C12   777777777777               Trailer VIN (2)
    TRLR3VIN                         C12   777777777777               Trailer VIN (3)
    MCARR_ID                         C11   00000000000                Motor Carrier Identification Number
    MCARR_I2                         C9    000000000                  MCID Identification Number
    VIN_1                            C1    3                          VIN Character(1)
    VIN_2                            C1    G                          VIN Character(2)
    VIN_3                            C1    N                          VIN Character(3)
    VIN_4                            C1    B                          VIN Character(4)
    VIN_5                            C1    A                          VIN Character(5)
    VIN_6                            C1    B                          VIN Character(6)
    VIN_7                            C1    F                          VIN Character(7)
    VIN_8                            C1    W                          VIN Character(8)
    VIN_9                            C1    X                          VIN Character(9)
    VIN_10                           C1    B                          VIN Character(10)
    VIN_11                           C1    S                          VIN Character(11)
    VIN_12                           C1    5                          VIN Character(12)
    TOTOBS                           C16   182                        TOTOBS


     -- NUMERIC --
    STATE                            N8              26               State Number
    ST_CASE                          N8          261018               Consecutive Number
    VEH_NO                           N8               1               Vehicle Number
    VE_FORMS                         N8               2               Number of Vehicle Forms Submitted for MV
    MONTH                            N8              10               Crash Date (Month)
    DAY                              N8              16               Crash Date (Day)
    HOUR                             N8               9               Crash Time (Hour)
    MINUTE                           N8              39               Crash Time (Minute)
    HARM_EV                          N8              12               First Harmful Event
    MAN_COLL                         N8               1               Manner of Collision of the First Harmful
    NUMOCCS                          N8               1               Number of Occupants
    UNITTYPE                         N8               1               Unit Type
    HIT_RUN                          N8               0               Hit and Run
    REG_STAT                         N8              26               Vehicle Registration State
    OWNER                            N8               2               Registered Vehicle Owner
    MOD_YEAR                         N8            2011               Vehicle Model Year
    VPICMAKE                         N8             467               vPIC Make
    VPICMODEL                        N8            4586               vPIC Model
    VPICBODYCLASS                    N8               7               vPIC Body Class
    MAKE                             N8              20               NCSA Make
    MODEL                            N8              23               NCSA Model
    BODY_TYP                         N8               6               NCSA Body Type
    ICFINALBODY                      N8               0               Final Stage Body Class
    GVWR_FROM                        N8              11               Power Unit Gross Vehicle Weight Rating F
    GVWR_TO                          N8              11               Power Unit Gross Vehicle Weight Rating T
    TOW_VEH                          N8               0               Vehicle Trailing
    TRLR1GVWR                        N8              77               Trailer Gross Vehicle Weight Rating (1)
    TRLR2GVWR                        N8              77               Trailer Gross Vehicle Weight Rating (2)
    TRLR3GVWR                        N8              77               Trailer Gross Vehicle Weight Rating (3)
    J_KNIFE                          N8               0               Jackknife
    MCARR_I1                         N8               0               MCID Issuing Authority
    V_CONFIG                         N8               0               Vehicle Configuration
    CARGO_BT                         N8               0               Cargo Body Type
    HAZ_INV                          N8               1               Hazardous Materials Involvement/Placard-
    HAZ_PLAC                         N8               0               Hazardous Materials Involvement/Placard-
    HAZ_ID                           N8               0               Hazardous Materials Involvement/Placard-
    HAZ_CNO                          N8               0               Hazardous Materials Involvement/Placard-
    HAZ_REL                          N8               0               Hazardous Materials Involvement/Placard-
    BUS_USE                          N8               0               Bus Use
    SPEC_USE                         N8               0               Special Use
    EMER_USE                         N8               0               Emergency Use
    TRAV_SP                          N8             998               Travel Speed
    UNDEROVERRIDE                    N8               8               Vehicle Underride/Override
    ROLLOVER                         N8               0               Rollover
    ROLINLOC                         N8               0               Location of Rollover
    IMPACT1                          N8              12               Areas of Impact - Initial Contact Point
    DEFORMED                         N8               6               Extent of Damage
    TOWED                            N8               6               Vehicle Towed
    M_HARM                           N8              12               Most Harmful Event
    FIRE_EXP                         N8               0               Fire Occurrence
    MAK_MOD                          N8           20023               NCSA Make Model Combined
    DEATHS                           N8               1               Fatals in Vehicle
    DR_DRINK                         N8               0               Driver Drinking
    DR_PRES                          N8               1               Driver Presence
    L_STATE                          N8              26               Driver License State
    DR_ZIP                           N8           48021               Driver ZIP Code
    L_TYPE                           N8               1               Non-CDL License Type
    L_STATUS                         N8               1               Non-CDL License Status
    CDL_STAT                         N8               0               Commercial MV License Status
    L_ENDORS                         N8               0               Compliance with CDL Endorsements
    L_COMPL                          N8               2               License Compliance with Class of Vehicle
    L_RESTRI                         N8               0               Compliance with License Restrictions
    DR_HGT                           N8              67               Driver Height
    DR_WGT                           N8             155               Driver Weight
    PREV_ACC                         N8               4               Previous Recorded Crashes
    PREV_SUS1                        N8               0               Previous Underage Administrative Per Se
    PREV_SUS2                        N8               0               Previous Administrative Per Se for BAC (
    PREV_SUS3                        N8               0               Previous Recorded Other Suspensions, Rev
    PREV_DWI                         N8               0               Previous DWI Convictions
    PREV_SPD                         N8               3               Previous Speeding Convictions
    PREV_OTH                         N8               3               Previous Other Moving Violation Convicti
    FIRST_MO                         N8               4               Date of Oldest Crash, Suspension or Conv
    FIRST_YR                         N8            2019               Date of Oldest Crash, Suspension or Conv
    LAST_MO                          N8               5               Date of Most Recent Crash, Suspension or
    LAST_YR                          N8            2023               Date of Most Recent Crash, Suspension or
    SPEEDREL                         N8               0               Speeding Related
    VTRAFWAY                         N8               2               Trafficway Description
    VNUM_LAN                         N8               3               Total Lanes in Roadway
    VSPD_LIM                         N8              45               Speed Limit
    VALIGN                           N8               1               Roadway Alignment
    VPROFILE                         N8               8               Roadway Grade
    VPAVETYP                         N8               8               Roadway Surface Type
    VSURCOND                         N8               1               Roadway Surface Condition
    VTRAFCON                         N8              97               Traffic Control Device
    VTCONT_F                         N8               8               Traffic Control Device Functioning
    P_CRASH1                         N8               1               Pre-Event Movement
    P_CRASH2                         N8              51               Critical Event - Precrash  (Event)
    P_CRASH3                         N8              99               Attempted Avoidance Maneuver
    PCRASH4                          N8               1               Pre- Impact Stability
    PCRASH5                          N8               1               Pre- Impact Location
    ACC_TYPE                         N8              24               Crash Type
    ACC_CONFIG                       N8             202               Crash Type Configuration

    /*              _     _            _     _        _     _
      __ _  ___ ___(_) __| | ___ _ __ | |_  | |_ __ _| |__ | | ___
     / _` |/ __/ __| |/ _` |/ _ \ `_ \| __| | __/ _` | `_ \| |/ _ \
    | (_| | (_| (__| | (_| |  __/ | | | |_  | || (_| | |_) | |  __/
     \__,_|\___\___|_|\__,_|\___|_| |_|\__|  \__\__,_|_.__/|_|\___|

    */

    Middle Observation(18827 ) of table = efars.accident - Total Obs 182 06FEB2026:08:29:52


     -- CHARACTER --
    Variable                        Typ    Value                      Label

    STATENAME                        C20   Minnesota                  State Name
    COUNTYNAME                       C105  SHERBURNE (141)            County Name
    CITYNAME                         C165  NOT APPLICABLE             City Name
    TWAY_ID                          C30   CR-4 97TH ST SE            Trafficway Identifier (1)
    TWAY_ID2                         C30   CR-11 165TH AVE SE         Trafficway Identifier (2)
    RAIL                             C7    0000000                    Rail Grade Crossing Identifier
    TOTOBS                           C16   182                        TOTOBS


     -- NUMERIC --

    FATALS                           N8               1               Fatalities

    STATE                            N8              27               State Number
    ST_CASE                          N8          270059               Consecutive Number
    PEDS                             N8               0               Number of Forms Submitted for Persons No
    PERNOTMVIT                       N8               0               Number of Persons Not in Motor Vehicles
    VE_TOTAL                         N8               2               Number of Vehicle Forms Submitted
    VE_FORMS                         N8               2               Number of Vehicle Forms Submitted for MV
    PVH_INVL                         N8               0               Number of Parked/Working Vehicles
    PERSONS                          N8               5               Number of MV Occupant
    PERMVIT                          N8               5               Number of Persons in Motor Vehicles In-T
    COUNTY                           N8             141               County
    CITY                             N8               0               City
    MONTH                            N8               5               Crash Date (Month)
    DAY                              N8               3               Crash Date (Day)
    DAY_WEEK                         N8               4               Crash Date (Day of Week)
    YEAR                             N8            2023               Crash Date (Year)
    HOUR                             N8              21               Crash Time (Hour)
    MINUTE                           N8               9               Crash Time (Minute)
    ROUTE                            N8               4               Route Signing
    RUR_URB                          N8               1               Rural Urban Classification
    FUNC_SYS                         N8               4               Functional System
    RD_OWNER                         N8               2               Ownership
    NHS                              N8               0               National Highway System
    SP_JUR                           N8               0               Special Jurisdiction
    MILEPT                           N8              28               MilePoint
    LATITUDE                         N8       45.429675               Global Position (Latitude)
    LONGITUD                         N8    -93.81923333               Global Position (Longitude)
    HARM_EV                          N8              12               First Harmful Event
    MAN_COLL                         N8               6               Manner of Collision of the First Harmful
    RELJCT1                          N8               0               Relation to Junction - Within Interchang
    RELJCT2                          N8               2               Relation to Junction - Specific Location
    TYP_INT                          N8               2               Type of Intersection
    REL_ROAD                         N8               1               Relation To Trafficway
    WRK_ZONE                         N8               0               Work Zone
    LGT_COND                         N8               3               Light Condition
    WEATHER                          N8               1               Atmospheric Condition
    SCH_BUS                          N8               0               School Bus Related
    NOT_HOUR                         N8              21               Notification Time EMS (Hour)
    NOT_MIN                          N8               9               Notification Time EMS (Min)
    ARR_HOUR                         N8              21               Arrival Time EMS (Hour)
    ARR_MIN                          N8              30               Arrival Time EMS (Min)
    HOSP_HR                          N8              22               EMS Time at Hospital (Hour)
    HOSP_MN                          N8              30               EMS Time at Hospital (Min)

    /*____        _ _   _        _     _                             _           _
    |___ /   __ _| | | | |_ __ _| |__ | | ___  ___    __ _ _ __   __| | ___ ___ | |_   _ _ __ ___  _ __  ___
      |_ \  / _` | | | | __/ _` | `_ \| |/ _ \/ __|  / _` | `_ \ / _` |/ __/ _ \| | | | | `_ ` _ \| `_ \/ __|
     ___) || (_| | | | | || (_| | |_) | |  __/\__ \ | (_| | | | | (_| | (_| (_) | | |_| | | | | | | | | \__ \
    |____/  \__,_|_|_|  \__\__,_|_.__/|_|\___||___/  \__,_|_| |_|\__,_|\___\___/|_|\__,_|_| |_| |_|_| |_|___/
    */


    Obs    TABLE                COLUMN                              LABEL

      1    MIACC                A1
      2    VPICDECODE           ACTIVESAFETYSYSNOTE                 Active Safety System Note
      3    DRUGS                DRUGACTQTY                          Actual Quantity
      4    VPICDECODE           ADAPTIVECRUISECONTROL               Adaptive Cruise Control (ACC)
      5    VPICDECODE           ADAPTIVECRUISECONTROLID             Adaptive Cruise Control (ACC) ID
      6    VPICDECODE           ADAPTIVEDRIVINGBEAM                 Adaptive Driving Beam (ADB)
      7    VPICDECODE           ADAPTIVEDRIVINGBEAMID               Adaptive Driving Beam (ADB) ID
      8    PBTYPE               PBAGE                               Age
      9    PERSON               AIR_BAG                             Air Bag Deployed
     10    PERSON               ALC_RES                             Alcohol Test- Result
     11    PERSON               ALC_STATUS                          Alcohol Test- Status
     12    PERSON               ATST_TYP                            Alcohol Test- Type
     13    VPICDECODE           ANTILOCKBRAKESYSTEM                 Anti-lock Braking System (ABS)
     14    VPICDECODE           ANTILOCKBRAKESYSTEMID               Anti-lock Braking System (ABS) ID
     15    VSOE                 AOI                                 Areas of Impact
     16    CEVENT               AOI2                                Areas of Impact (Other Vehicle)
     17    CEVENT               AOI1                                Areas of Impact (This Vehicle)
     18    DAMAGE               DAMAGE                              Areas of Impact - Damaged Areas
     19    PARKWORK             PIMPACT1                            Areas of Impact - Initial Contact Point
     20    ACCIDENT             ARR_HOUR                            Arrival Time EMS (Hour)
     21    ACCIDENT             ARR_MIN                             Arrival Time EMS (Min)
     22    ACCIDENT             WEATHER                             Atmospheric Condition
     23    VEHICLE              P_CRASH3                            Attempted Avoidance Maneuver
     24    VPICDECODE           AUTOREVERSESYSTEM                   Auto-Reverse System for Windows and Sunroofs
     25    VPICDECODE           AUTOREVERSESYSTEMID                 Auto-Reverse System for Windows and Sunroofs ID
     26    VPICDECODE           AUTOMATICCRASHNOTIFICATION          Automatic Crash Notification (ACN) / Advanced Automatic Crash Notification (AACN)
     27    VPICDECODE           AUTOMATICCRASHNOTIFICATIONID        Automatic Crash Notification (ACN) / Advanced Automatic Crash Notification (AACN) ID
     28    VPICDECODE           AUTOPEDESTRIANALERTINGSOUND         Automatic Pedestrian Alerting Sound (for Hybrid and EV only)
     29    VPICDECODE           AUTOPEDESTRIANALERTINGSOUNDID       Automatic Pedestrian Alerting Sound (for Hybrid and EV only) ID
     30    VPICDECODE           AXLECONFIGURATION                   Axle Configuration
     31    VPICDECODE           AXLECONFIGURATIONID                 Axle Configuration ID
     32    VPICDECODE           AXLESCOUNT                          Axles
     33    VPICDECODE           BACKUPCAMERA                        Backup Camera
     34    VPICDECODE           BACKUPCAMERAID                      Backup Camera ID
     35    VPICDECODE           BASEPRICE                           Base Price ($)
     36    VPICDECODE           BATTERYA_FROM                       Battery Current (Amps) From
     37    VPICDECODE           BATTERYA_TO                         Battery Current (Amps) To
     38    VPICDECODE           BATTERYKWH_FROM                     Battery Energy (kWh) From
     39    VPICDECODE           BATTERYKWH_TO                       Battery Energy (kWh) To
     40    VPICDECODE           OTHERBATTERYINFO                    Battery Info
     41    VPICDECODE           BATTERYTYPE                         Battery Type
     42    VPICDECODE           BATTERYTYPEID                       Battery Type ID
     43    VPICDECODE           BATTERYV_FROM                       Battery Voltage (Volts) From
     44    VPICDECODE           BATTERYV_TO                         Battery Voltage (Volts) To
     45    VPICDECODE           TRUCKBEDLENGTHIN                    Bed Length (inches)
     46    VPICDECODE           TRUCKBEDTYPE                        Bed Type
     47    VPICDECODE           TRUCKBEDTYPEID                      Bed Type ID
     48    PBTYPE               BIKECGP                             Bicyclist Crash Group
     49    PBTYPE               BIKELOC                             Bicyclist Crash Location
     50    PBTYPE               BIKECTYPE                           Bicyclist Crash Type
     51    PBTYPE               BIKEDIR                             Bicyclist Direction
     52    PBTYPE               BIKEPOS                             Bicyclist Position
     53    VPICDECODE           BLINDSPOTINTERVENTION               Blind Spot Intervention (BSI)
     54    VPICDECODE           BLINDSPOTINTERVENTIONID             Blind Spot Intervention (BSI) ID
     55    VPICDECODE           BLINDSPOTWARNING                    Blind Spot Warning (BSW)
     56    VPICDECODE           BLINDSPOTWARNINGID                  Blind Spot Warning (BSW) ID
     57    VPICDECODE           BODYCLASS                           Body Class
     58    VPICDECODE           BODYCLASSID                         Body Class ID
     59    VPICDECODE           BRAKESYSTEMDESC                     Brake System Description
     60    VPICDECODE           BRAKESYSTEMTYPE                     Brake System Type
     61    VPICDECODE           BRAKESYSTEMTYPEID                   Brake System Type ID
     62    VPICDECODE           BUSFLOORCONFIGURATIONTYPE           Bus Floor Configuration Type
     63    VPICDECODE           BUSFLOORCONFIGURATIONTYPEID         Bus Floor Configuration Type ID
     64    VPICDECODE           BUSLENGTHFT                         Bus Length (feet)
     65    VPICDECODE           BUSTYPE                             Bus Type
     66    VPICDECODE           BUSTYPEID                           Bus Type ID
     67    PARKWORK             PBUS_USE                            Bus Use
     68    VPICDECODE           TRUCKBODYCABTYPE                    Cab Type
     69    VPICDECODE           TRUCKBODYCABTYPEID                  Cab Type ID
     70    PARKWORK             PCARGTYP                            Cargo Body Type
     71    VPICDECODE           CHARGERLEVEL                        Charger Level
     72    VPICDECODE           CHARGERLEVELID                      Charger Level ID
     73    VPICDECODE           CHARGERPOWERKW                      Charger Power (kW)
     74    ACCIDENT             CITY                                City
     75    ACCIDENT             CITYNAME                            City Name
     76    VEHICLE              CDL_STAT                            Commercial MV License Status
     77    VEHICLE              L_ENDORS                            Compliance with CDL Endorsements
     78    VEHICLE              L_RESTRI                            Compliance with License Restrictions
     79    DRIMPAIR             DRIMPAIR                            Condition (Impairment) at Time of Crash
     80    ACCIDENT             ST_CASE                             Consecutive Number
     81    FACTOR               VEHICLECC                           Contributing Circumstances, Motor Vehicle
     82    VPICDECODE           ENGINECOOLINGTYPE                   Cooling Type
     83    VPICDECODE           ENGINECOOLINGTYPEID                 Cooling Type ID
     84    ACCIDENT             COUNTY                              County
     85    ACCIDENT             COUNTYNAME                          County Name
     86    ACCIDENT             DAY_WEEK                            Crash Date (Day of Week)
     87    ACCIDENT             DAY                                 Crash Date (Day)
     88    ACCIDENT             MONTH                               Crash Date (Month)
     89    ACCIDENT             YEAR                                Crash Date (Year)
     90    VPICDECODE           CRASHIMMINENTBRAKING                Crash Imminent Braking (CIB)
     91    VPICDECODE           CRASHIMMINENTBRAKINGID              Crash Imminent Braking (CIB) ID
     92    CRASHRF              CRASHRF                             Crash Related Factor
     93    ACCIDENT             HOUR                                Crash Time (Hour)
     94    ACCIDENT             MINUTE                              Crash Time (Minute)
     95    VEHICLE              ACC_TYPE                            Crash Type
     96    VEHICLE              ACC_CONFIG                          Crash Type Configuration
     97    PERSON               LAG_MINS                            Crash to Death Minutes (Minutes)
     98    PERSON               LAG_HRS                             Crash to Death Time (Hours)
     99    VEHICLE              P_CRASH2                            Critical Event - Precrash  (Event)
    100    VPICDECODE           CURBWEIGHTLB                        Curb Weight (pounds)
    101    VPICDECODE           AIRBAGLOCCURTAIN                    Curtain Air Bag Locations
    102    VPICDECODE           AIRBAGLOCCURTAINID                  Curtain Air Bag Locations ID
    103    VPICDECODE           CUSTOMMOTORCYCLETYPE                Custom Motorcycle Type
    104    VPICDECODE           CUSTOMMOTORCYCLETYPEID              Custom Motorcycle Type ID
    105    VEHICLE              LAST_MO                             Date of Most Recent Crash, Suspension or Conviction (Month)
    106    VEHICLE              LAST_YR                             Date of Most Recent Crash, Suspension or Conviction (Year)
    107    VEHICLE              FIRST_MO                            Date of Oldest Crash, Suspension or Conviction (Month)
    108    VEHICLE              FIRST_YR                            Date of Oldest Crash, Suspension or Conviction (Year)
    109    VPICDECODE           DAYTIMERUNNINGLIGHT                 Daytime Running Light (DRL)
    110    VPICDECODE           DAYTIMERUNNINGLIGHTID               Daytime Running Light (DRL) ID
    111    PERSON               DEATH_DA                            Death Date (Day)
    112    PERSON               DEATH_MO                            Death Date (Month)
    113    PERSON               DEATH_YR                            Death Date (Year)
    114    PERSON               DEATH_TM                            Death Time
    115    PERSON               DEATH_HR                            Death Time (Hour)
    116    PERSON               DEATH_MN                            Death Time (Minutes)
    117    VPICDECODE           DESTINATIONMARKET                   Destination Market
    118    VPICDECODE           DESTINATIONMARKETID                 Destination Market ID
    119    PERSON               DOA                                 Died at Scene/En Route
    120    VPICDECODE           DISPLACEMENTCC                      Displacement (CC)
    121    VPICDECODE           DISPLACEMENTCI                      Displacement (CI)
    122    VPICDECODE           DISPLACEMENTL                       Displacement (L)
    123    VPICDECODE           DOORSCOUNT                          Doors
    124    VPICDECODE           DRIVETYPE                           Drive Type
    125    VPICDECODE           DRIVETYPEID                         Drive Type ID
    126    DISTRACT             DRDISTRACT                          Driver Distracted By
    127    VEHICLE              DR_DRINK                            Driver Drinking
    128    VEHICLE              DR_HGT                              Driver Height
    129    VEHICLE              L_STATE                             Driver License State
    130    MANEUVER             MANEUVER                            Driver Maneuvered To Avoid
    131    VEHICLE              DR_PRES                             Driver Presence
    132    DRIVERRF             DRIVERRF                            Driver Related Factor
    133    VEHICLE              DR_WGT                              Driver Weight
    134    VEHICLE              DR_ZIP                              Driver ZIP Code
    135    VISION               VISION                              Driver(s) Vision Obscured By
    136    DRUGS                DRUGQTY                             Drug Quantity
    137    DRUGS                DRUGSPEC                            Drug Specimen
    138    PERSON               DSTATUS                             Drug Test -Status
    139    DRUGS                DRUGRES                             Drug Test Result
    140    DRUGS                DRUGMETHOD                          Drug Testing Method
    141    VPICDECODE           DYNAMICBRAKESUPPORT                 Dynamic Brake Support (DBS)
    142    VPICDECODE           DYNAMICBRAKESUPPORTID               Dynamic Brake Support (DBS) ID
    143    ACCIDENT             HOSP_HR                             EMS Time at Hospital (Hour)
    144    ACCIDENT             HOSP_MN                             EMS Time at Hospital (Min)
    145    VPICDECODE           EVDRIVEUNIT                         EV Drive Unit
    146    VPICDECODE           EVDRIVEUNITID                       EV Drive Unit ID
    147    PERSON               EJECTION                            Ejection
    148    PERSON               EJ_PATH                             Ejection Path
    149    VPICDECODE           ENGINEELECTRIFICATIONLEVEL          Electrification Level
    150    VPICDECODE           ENGINEELECTRIFICATIONLEVELID        Electrification Level ID
    151    VPICDECODE           ELECTRONICSTABILITYCONTROL          Electronic Stability Control (ESC)
    152    VPICDECODE           ELECTRONICSTABILITYCONTROLID        Electronic Stability Control (ESC) ID
    153    PARKWORK             PEM_USE                             Emergency Use
    154    VPICDECODE           ENGINEBRAKEHP_FROM                  Engine Brake (hp) From
    155    VPICDECODE           ENGINEBRAKEHP_TO                    Engine Brake (hp) To
    156    VPICDECODE           ENGINECONFIGURATION                 Engine Configuration
    157    VPICDECODE           ENGINECONFIGURATIONID               Engine Configuration ID
    158    VPICDECODE           ENGINEMANUFACTURER                  Engine Manufacturer
    159    VPICDECODE           ENGINEMODEL                         Engine Model
    160    VPICDECODE           ENGINECYLINDERSCOUNT                Engine Number of Cylinders
    161    VPICDECODE           ENGINEPOWERKW                       Engine Power (kW)
    162    VPICDECODE           ENGINESTROKECYCLES                  Engine Stroke Cycles
    163    VPICDECODE           ENTERTAINMENTSYSTEM                 Entertainment System
    164    VPICDECODE           ENTERTAINMENTSYSTEMID               Entertainment System ID
    165    VPICDECODE           VINDECODEERROR                      Error Code
    166    VPICDECODE           EVENTDATARECORDER                   Event Data Recorder (EDR)
    167    VPICDECODE           EVENTDATARECORDERID                 Event Data Recorder (EDR) ID
    168    CEVENT               EVENTNUM                            Event Number
    169    PARKWORK             PVEH_SEV                            Extent of Damage
    170    PERSON               EXTRICAT                            Extrication
    171    PERSON               WORK_INJ                            Fatal Injury at Work
    172    ACCIDENT             FATALS                              Fatalities
    173    PARKWORK             PDEATHS                             Fatals in Vehicle
    174    PARKWORK             PICFINALBODY                        Final Stage Body Class
    175    PARKWORK             PFIRE                               Fire Occurrence
    176    ACCIDENT             HARM_EV                             First Harmful Event
    177    VPICDECODE           FORWARDCOLLISIONWARNING             Forward Collision Warning (FCW)
    178    VPICDECODE           FORWARDCOLLISIONWARNINGID           Forward Collision Warning (FCW) ID
    179    VPICDECODE           AIRBAGLOCFRONT                      Front Air Bag Locations
    180    VPICDECODE           AIRBAGLOCFRONTID                    Front Air Bag Locations ID
    181    VPICDECODE           FUELDELIVERYINJECTIONTYPE           Fuel Delivery / Fuel Injection Type
    182    VPICDECODE           FUELDELIVERYINJECTIONTYPEID         Fuel Delivery / Fuel Injection Type ID
    183    VPICDECODE           FUELTYPEPRIMARY                     Fuel Type - Primary
    184    VPICDECODE           FUELTYPEPRIMARYID                   Fuel Type - Primary ID
    185    VPICDECODE           FUELTYPESECONDARY                   Fuel Type - Secondary
    186    VPICDECODE           FUELTYPESECONDARYID                 Fuel Type - Secondary ID
    187    ACCIDENT             FUNC_SYS                            Functional System
    188    ACCIDENT             LATITUDE                            Global Position (Latitude)
    189    ACCIDENT             LONGITUD                            Global Position (Longitude)
    190    VPICDECODE           GROSSCOMBWEIGHTRATINGFROM           Gross Combination Weight Rating From
    191    VPICDECODE           GROSSCOMBWEIGHTRATINGFROMID         Gross Combination Weight Rating From ID
    192    VPICDECODE           GROSSCOMBWEIGHTRATINGTO             Gross Combination Weight Rating To
    193    VPICDECODE           GROSSCOMBWEIGHTRATINGTOID           Gross Combination Weight Rating To ID
    194    VPICDECODE           GROSSVEHICLEWEIGHTRATINGFROM        Gross Vehicle Weight Rating From
    195    VPICDECODE           GROSSVEHICLEWEIGHTRATINGFROMID      Gross Vehicle Weight Rating From ID
    196    VPICDECODE           GROSSVEHICLEWEIGHTRATINGTO          Gross Vehicle Weight Rating To
    197    VPICDECODE           GROSSVEHICLEWEIGHTRATINGTOID        Gross Vehicle Weight Rating To ID
    198    PARKWORK             PHAZ_ID                             Hazardous Materials Involvement/Placard- HM3 Identification Number
    199    PARKWORK             PHAZ_INV                            Hazardous Materials Involvement/Placard-HM1 -Involvement
    200    PARKWORK             PHAZPLAC                            Hazardous Materials Involvement/Placard-HM2 Placard
    201    PARKWORK             PHAZ_CNO                            Hazardous Materials Involvement/Placard-HM4 Class Number
    202    PARKWORK             PHAZ_REL                            Hazardous Materials Involvement/Placard-HM5 Released
    203    VPICDECODE           HEADLAMPLIGHTSOURCE                 Headlamp Light Source
    204    VPICDECODE           HEADLAMPLIGHTSOURCEID               Headlamp Light Source ID
    205    PERSON               HELM_USE                            Helmet Use
    206    PERSON               HISPANIC                            Hispanic Origin
    207    PARKWORK             PHIT_RUN                            Hit and Run
    208    PERSON               HELM_MIS                            Indication of Helmet Misuse?
    209    PERSON               REST_MIS                            Indication of Restraint System Misuse?
    210    PERSON               INJ_SEV                             Injury Severity
    211    RACE                 MULTRACE                            Is Multiple Race?
    212    VEHICLE              J_KNIFE                             Jackknife
    213    VPICDECODE           KEYLESSIGNITION                     Keyless Ignition
    214    VPICDECODE           KEYLESSIGNITIONID                   Keyless Ignition ID
    215    VPICDECODE           AIRBAGLOCKNEE                       Knee Air Bag Locations
    216    VPICDECODE           AIRBAGLOCKNEEID                     Knee Air Bag Locations ID
    217    VPICDECODE           LANECENTERINGASSISTANCE             Lane Centering Assistance
    218    VPICDECODE           LANECENTERINGASSISTANCEID           Lane Centering Assistance ID
    219    VPICDECODE           LANEDEPARTUREWARNING                Lane Departure Warning (LDW)
    220    VPICDECODE           LANEDEPARTUREWARNINGID              Lane Departure Warning (LDW) ID
    221    VPICDECODE           LANEKEEPINGASSISTANCE               Lane Keeping Assistance (LKA)
    222    VPICDECODE           LANEKEEPINGASSISTANCEID             Lane Keeping Assistance (LKA) ID
    223    PBTYPE               PEDLEG                              Leg Intersection
    224    VEHICLE              L_COMPL                             License Compliance with Class of Vehicle
    225    ACCIDENT             LGT_COND                            Light Condition
    226    VEHICLE              ROLINLOC                            Location of Rollover
    227    PARKWORK             PMCARR_I2                           MCID Identification Number
    228    PARKWORK             PMCARR_I1                           MCID Issuing Authority
    229    VPICDECODE           MAKE                                Make
    230    VPICDECODE           MAKEID                              Make ID
    231    ACCIDENT             MAN_COLL                            Manner of Collision of the First Harmful Event
    232    VPICDECODE           MANUFACTURERFULLNAME                Manufacturer Name
    233    VPICDECODE           MANUFACTURERFULLNAMEID              Manufacturer Name ID
    234    PBTYPE               PBCWALK                             Marked Crosswalk
    235    VPICDECODE           VEHICLEDESCRIPTOR                   Masked VIN
    236    ACCIDENT             MILEPT                              MilePoint
    237    VPICDECODE           MODEL                               Model
    238    VPICDECODE           MODELID                             Model ID
    239    VPICDECODE           MODELYEAR                           Model Year
    240    PARKWORK             PM_HARM                             Most Harmful Event
    241    PARKWORK             PMCARR_ID                           Motor Carrier Identification Number
    242    VPICDECODE           MOTORCYCLECHASSISTYPE               Motorcycle Chassis Type
    243    VPICDECODE           MOTORCYCLECHASSISTYPEID             Motorcycle Chassis Type ID
    244    VPICDECODE           MOTORCYCLESUSPENSIONTYPE            Motorcycle Suspension Type
    245    VPICDECODE           MOTORCYCLESUSPENSIONTYPEID          Motorcycle Suspension Type ID
    246    PBTYPE               MOTDIR                              Motorist Direction
    247    PBTYPE               MOTMAN                              Motorist Maneuver
    248    PARKWORK             PBODYTYP                            NCSA Body Type
    249    PARKWORK             PMAKE                               NCSA Make
    250    PARKWORK             PMAK_MOD                            NCSA Make Model Combined
    251    PARKWORK             PMODEL                              NCSA Model
    252    ACCIDENT             NHS                                 National Highway System
    253    VEHICLE              L_STATUS                            Non-CDL License Status
    254    VEHICLE              L_TYPE                              Non-CDL License Type
    255    VPICDECODE           NONLANDUSE                          Non-Land Use
    256    VPICDECODE           NONLANDUSEID                        Non-Land Use ID
    257    NMPRIOR              NMACTION                            Non-Motorist Action/Circumstances
    258    NMCRASH              NMCC                                Non-Motorist Contributing Circumstances
    259    PERSON               DEVMOTOR                            Non-Motorist Device Motorization
    260    PERSON               DEVTYPE                             Non-Motorist Device Type
    261    NMDISTRACT           NMDISTRACT                          Non-Motorist Distracted By
    262    SAFETYEQ             NMHELMET                            Non-Motorist Helmet Use
    263    PERSON               LOCATION                            Non-Motorist Location at Time of Crash
    264    SAFETYEQ             NMLIGHT                             Non-Motorist Use of Lighting
    265    SAFETYEQ             NMOTHPRE                            Non-Motorist Use of Other Preventive Safety Equipment
    266    SAFETYEQ             NMOTHPRO                            Non-Motorist Use of Other Protective Safety Equipment
    267    SAFETYEQ             NMPROPAD                            Non-Motorist Use of Protective Pads
    268    SAFETYEQ             NMREFCLO                            Non-Motorist Use of Reflective Clothing/Carried Item
    269    VPICDECODE           NOTE                                Note
    270    ACCIDENT             NOT_HOUR                            Notification Time EMS (Hour)
    271    ACCIDENT             NOT_MIN                             Notification Time EMS (Min)
    272    VPICDECODE           BATTERYCELLSPERMODULE               Number of Battery Cells per Module
    273    VPICDECODE           BATTERYMODULESPERPACK               Number of Battery Modules per Pack
    274    VPICDECODE           BATTERYPACKSPERVEHICLE              Number of Battery Packs per Vehicle
    275    ACCIDENT             PEDS                                Number of Forms Submitted for Persons Not in Motor Vehicles
    276    ACCIDENT             PERSONS                             Number of MV Occupant
    212    VEHICLE              J_KNIFE                             Jackknife
    278    ACCIDENT             PVH_INVL                            Number of Parked/Working Vehicles
    279    ACCIDENT             PERNOTMVIT                          Number of Persons Not in Motor Vehicles In-Transport
    280    ACCIDENT             PERMVIT                             Number of Persons in Motor Vehicles In-Transport
    281    VPICDECODE           SEATROWSCOUNT                       Number of Seat Rows
    282    VPICDECODE           SEATSCOUNT                          Number of Seats
    283    ACCIDENT             VE_TOTAL                            Number of Vehicle Forms Submitted
    284    ACCIDENT             VE_FORMS                            Number of Vehicle Forms Submitted for MV In Transport
    285    VPICDECODE           WHEELSCOUNT                         Number of Wheels
    286    RACE                 ORDER                               Order
    287    VPICDECODE           OTHERBUSINFO                        Other Bus Info
    288    VPICDECODE           OTHERENGINEINFO                     Other Engine Info
    289    VPICDECODE           OTHERMOTORCYCLEINFO                 Other Motorcycle Info
    290    VPICDECODE           OTHERRESTRAINTSYSTEMINFO            Other Restraint System Info
    291    VPICTRAILERDECODE    OTHERTRAILERINFO                    Other Trailer Info
    292    ACCIDENT             RD_OWNER                            Ownership
    293    VPICDECODE           PARKASSIST                          Parking Assist
    294    VPICDECODE           PARKASSISTID                        Parking Assist ID
    295    VPICDECODE           PEDESTRIANAUTOEMERGENCYBRAKING      Pedestrian Automatic Emergency Braking (PAEB)
    296    VPICDECODE           PEDESTRIANAUTOEMERGENCYBRAKINGID    Pedestrian Automatic Emergency Braking (PAEB) ID
    297    PBTYPE               PEDCGP                              Pedestrian Crash Group
    298    PBTYPE               PEDLOC                              Pedestrian Crash Location
    299    PBTYPE               PEDCTYPE                            Pedestrian Crash Type
    300    PBTYPE               PEDDIR                              Pedestrian Direction
    301    PBTYPE               PEDPOS                              Pedestrian Position
    302    PBTYPE               PEDSNR                              Pedestrian Scenario
    303    DRUGS                PER_NO                              Person Number
    304    PERSONRF             PERSONRF                            Person Related Factor
    305    PBTYPE               PBPTYPE                             Person Type
    306    VPICDECODE           PLANTCITY                           Plant City
    307    VPICDECODE           PLANTCOMPANYNAME                    Plant Company Name
    308    VPICDECODE           PLANTCOUNTRY                        Plant Country
    309    VPICDECODE           PLANTCOUNTRYID                      Plant Country ID
    310    VPICDECODE           PLANTSTATE                          Plant State
    311    PERSON               DRINKING                            Police Reported Alcohol Involvement
    312    PERSON               DRUGS                               Police Reported Drug Involvement
    313    PARKWORK             PGVWR_FROM                          Power Unit Gross Vehicle Weight Rating From
    314    PARKWORK             PGVWR_TO                            Power Unit Gross Vehicle Weight Rating To
    315    VEHICLE              PCRASH5                             Pre- Impact Location
    316    VEHICLE              PCRASH4                             Pre- Impact Stability
    317    VEHICLE              P_CRASH1                            Pre-Event Movement
    318    VPICDECODE           PRETENSIONER                        Pretensioner
    319    VPICDECODE           PRETENSIONERID                      Pretensioner ID
    320    VEHICLE              PREV_SUS2                           Previous Administrative Per Se for BAC (Not Underage)
    321    VEHICLE              PREV_DWI                            Previous DWI Convictions
    322    VEHICLE              PREV_OTH                            Previous Other Moving Violation Convictions
    323    VEHICLE              PREV_ACC                            Previous Recorded Crashes
    324    VEHICLE              PREV_SUS3                           Previous Recorded Other Suspensions, Revocations, or Withdrawals
    325    VEHICLE              PREV_SPD                            Previous Speeding Convictions
    326    VEHICLE              PREV_SUS1                           Previous Underage Administrative Per Se for BAC
    327    RACE                 RACE                                Race
    328    ACCIDENT             RAIL                                Rail Grade Crossing Identifier
    329    VPICDECODE           REARAUTOMATICEMERGENCYBRAKING       Rear Automatic Emergency Braking
    330    VPICDECODE           REARAUTOMATICEMERGENCYBRAKINGID     Rear Automatic Emergency Braking ID
    331    VPICDECODE           REARCROSSTRAFFICALERT               Rear Cross Traffic Alert
    332    VPICDECODE           REARCROSSTRAFFICALERTID             Rear Cross Traffic Alert ID
    333    PARKWORK             POWNER                              Registered Vehicle Owner
    334    ACCIDENT             REL_ROAD                            Relation To Trafficway
    335    ACCIDENT             RELJCT2                             Relation to Junction - Specific Location
    336    ACCIDENT             RELJCT1                             Relation to Junction - Within Interchange Area
    337    VEHICLE              VALIGN                              Roadway Alignment
    338    VEHICLE              VPROFILE                            Roadway Grade
    339    VEHICLE              VSURCOND                            Roadway Surface Condition
    340    VEHICLE              VPAVETYP                            Roadway Surface Type
    341    PERSON               ROLLOVER                            Rollover
    342    ACCIDENT             ROUTE                               Route Signing
    343    ACCIDENT             RUR_URB                             Rural Urban Classification
    344    VPICDECODE           SAEAUTOMATIONLEVEL_FROM             SAE Automation Level From
    345    VPICDECODE           SAEAUTOMATIONLEVEL_TO               SAE Automation Level To
    346    ACCIDENT             SCH_BUS                             School Bus Related
    347    PBTYPE               PBSZONE                             School Zone
    348    VPICDECODE           SEATBELTTYPE                        Seat Belt Type
    349    VPICDECODE           SEATBELTTYPEID                      Seat Belt Type ID
    350    VPICDECODE           AIRBAGLOCSEATCUSHION                Seat Cushion Air Bag Locations
    351    VPICDECODE           AIRBAGLOCSEATCUSHIONID              Seat Cushion Air Bag Locations ID
    352    PERSON               SEAT_POS                            Seating Position
    353    VPICDECODE           SEMIAUTOHEADLAMPBEAMSWITCHING       Semiautomatic Headlamp Beam Switching
    354    VPICDECODE           SEMIAUTOHEADLAMPBEAMSWITCHINGID     Semiautomatic Headlamp Beam Switching ID
    355    CEVENT               SOE                                 Sequence of Events
    356    VPICDECODE           SERIES                              Series
    357    VPICDECODE           SERIES2                             Series2
    358    PBTYPE               PBSEX                               Sex
    359    VPICDECODE           AIRBAGLOCSIDE                       Side Air Bag Locations
    360    VPICDECODE           AIRBAGLOCSIDEID                     Side Air Bag Locations ID
    361    PBTYPE               PBSWALK                             Sidewalk Presence
    362    ACCIDENT             SP_JUR                              Special Jurisdiction
    363    PARKWORK             PSP_USE                             Special Use
    364    VEHICLE              VSPD_LIM                            Speed Limit
    365    VEHICLE              SPEEDREL                            Speeding Related
    366    ACCIDENT             STATENAME                           State Name
    367    ACCIDENT             STATE                               State Number
    368    VPICDECODE           STEERINGLOCATION                    Steering Location
    369    VPICDECODE           STEERINGLOCATIONID                  Steering Location ID
    370    VPICDECODE           TPMS                                Tire Pressure Monitoring System (TPMS) Type
    371    VPICDECODE           TPMSID                              Tire Pressure Monitoring System (TPMS) Type ID
    372    VPICDECODE           TOPSPEEDMPH                         Top Speed (MPH)
    373    VEHICLE              VNUM_LAN                            Total Lanes in Roadway
    374    VPICDECODE           TRACKWIDTHIN                        Track Width (inches)
    375    VPICDECODE           TRACTIONCONTROL                     Traction Control
    376    VPICDECODE           TRACTIONCONTROLID                   Traction Control ID
    377    VEHICLE              VTRAFCON                            Traffic Control Device
    378    VEHICLE              VTCONT_F                            Traffic Control Device Functioning
    379    VEHICLE              VTRAFWAY                            Trafficway Description
    380    ACCIDENT             TWAY_ID                             Trafficway Identifier (1)
    381    ACCIDENT             TWAY_ID2                            Trafficway Identifier (2)
    382    VPICTRAILERDECODE    TRAILERBODYTYPE                     Trailer Body Type
    383    VPICTRAILERDECODE    TRAILERBODYTYPEID                   Trailer Body Type ID
    384    PARKWORK             PTRLR1GVWR                          Trailer Gross Vehicle Weight Rating (1)
    385    PARKWORK             PTRLR2GVWR                          Trailer Gross Vehicle Weight Rating (2)
    386    PARKWORK             PTRLR3GVWR                          Trailer Gross Vehicle Weight Rating (3)
    387    VPICTRAILERDECODE    TRAILERLENGTHFT                     Trailer Length (feet)
    388    VPICTRAILERDECODE    TRAILER_NO                          Trailer Number
    389    VPICTRAILERDECODE    TRAILERTYPECONNECTION               Trailer Type Connection
    390    VPICTRAILERDECODE    TRAILERTYPECONNECTIONID             Trailer Type Connection ID
    391    PARKWORK             PTRLR1VIN                           Trailer VIN (1)
    392    PARKWORK             PTRLR2VIN                           Trailer VIN (2)
    393    PARKWORK             PTRLR3VIN                           Trailer VIN (3)
    394    VPICDECODE           TRANSMISSIONSPEEDS                  Transmission Speeds
    395    VPICDECODE           TRANSMISSIONSTYLE                   Transmission Style
    396    VPICDECODE           TRANSMISSIONSTYLEID                 Transmission Style ID
    397    PERSON               HOSPITAL                            Transported to First Medical Facility By
    398    VEHICLE              TRAV_SP                             Travel Speed
    399    VPICDECODE           TRIM                                Trim
    400    VPICDECODE           TRIM2                               Trim2
    401    VPICDECODE           ENGINETURBO                         Turbo
    402    VPICDECODE           ENGINETURBOID                       Turbo ID
    403    ACCIDENT             TYP_INT                             Type of Intersection
    404    PERSON               REST_USE                            Type of Restraint System in Use
    405    PARKWORK             PTYPE                               Unit Type
    406    DRUGS                DRUGUOM                             Unit of Measure
    407    PARKWORK             PVIN_1                              VIN Character(1)
    408    PARKWORK             PVIN_10                             VIN Character(10)
    409    PARKWORK             PVIN_11                             VIN Character(11)
    410    PARKWORK             PVIN_12                             VIN Character(12)
    411    PARKWORK             PVIN_2                              VIN Character(2)
    412    PARKWORK             PVIN_3                              VIN Character(3)
    413    PARKWORK             PVIN_4                              VIN Character(4)
    414    PARKWORK             PVIN_5                              VIN Character(5)
    415    PARKWORK             PVIN_6                              VIN Character(6)
    416    PARKWORK             PVIN_7                              VIN Character(7)
    417    PARKWORK             PVIN_8                              VIN Character(8)
    418    PARKWORK             PVIN_9                              VIN Character(9)
    419    VPICDECODE           VINDECODEDON                        VIN Decoded On
    420    VPICDECODE           ENGINEVALVETRAINDESIGN              Valve Train Design
    421    VPICDECODE           ENGINEVALVETRAINDESIGNID            Valve Train Design ID
    422    PARKWORK             PV_CONFIG                           Vehicle Configuration
    423    VEVENT               VEVENTNUM                           Vehicle Event Number
    424    PARKWORK             PVIN                                Vehicle Identification Number
    425    PARKWORK             PMODYEAR                            Vehicle Model Year
    426    DAMAGE               VEH_NO                              Vehicle Number
    427    CEVENT               VNUMBER2                            Vehicle Number (Other Vehicle)
    428    CEVENT               VNUMBER1                            Vehicle Number (This Vehicle)
    429    PERSON               STR_VEH                             Vehicle Number Of Motor Vehicle Striking Non-Motorist
    430    PARKWORK             PREG_STAT                           Vehicle Registration State
    431    PVEHICLESF           PVEHICLESF                          Vehicle Related Factor
    432    PARKWORK             PTOWED                              Vehicle Towed
    433    PARKWORK             PTRAILER                            Vehicle Trailing
    434    VPICDECODE           VEHICLETYPE                         Vehicle Type
    435    VPICDECODE           VEHICLETYPEID                       Vehicle Type ID
    436    PARKWORK             PUNDEROVERRIDE                      Vehicle Underride/Override
    437    VIOLATN              VIOLATION                           Violations Charged
    438    VPICDECODE           WHEELBASEIN_FROM                    Wheel Base (inches) From
    439    VPICDECODE           WHEELBASEIN_TO                      Wheel Base (inches) To
    440    VPICDECODE           WHEELBASETYPE                       Wheel Base Type
    441    VPICDECODE           WHEELBASETYPEID                     Wheel Base Type ID
    442    VPICDECODE           WHEELSIZEFRONTIN                    Wheel Size Front (inches)
    443    VPICDECODE           WHEELSIZEREARIN                     Wheel Size Rear (inches)
    444    VPICDECODE           WINDOWS                             Windows
    445    ACCIDENT             WRK_ZONE                            Work Zone
    446    PARKWORK             PVPICBODYCLASS                      vPIC Body Class
    447    PARKWORK             PVPICMAKE                           vPIC Make
    448    PARKWORK             PVPICMODEL                          vPIC Model


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */







