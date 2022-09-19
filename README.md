# utl-python-r-and-sas-sql-solutions-to-add-missing-rows-to-a-data-table
Python R and sas sql solutions to add missing rows to a data table
    %let pgm=utl-python-r-and-sas-sql-solutions-to-add-missing-rows-to-a-data-table;

    Python R and sas sql solutions to add missing rows to a data table

    This repo
    https://tinyurl.com/9x9sp9vv
    https://github.com/rogerjdeangelis/utl-python-r-and-sas-sql-solutions-to-add-missing-rows-to-a-data-table

    Support for SQL is very poor in R and Python.
    For instance outer joins are not supported in R or Python and most stat/math functions are
    not directly supported in Python. unctions.

    StackOverflow R
    https://tinyurl.com/58mkv62d
    https://stackoverflow.com/questions/73765595/conditional-insertion-of-extra-row-in-data-table

    Solution by Akrun
    https://stackoverflow.com/users/3732271/akrun

    Related github
    https://tinyurl.com/khkhkxxm
    https://github.com/rogerjdeangelis/utl-sqlite-processing-in-python-with-added-math-and-stat-functions


    /*          _
     _ __ _   _| | ___  ___
    | `__| | | | |/ _ \/ __|
    | |  | |_| | |  __/\__ \
    |_|   \__,_|_|\___||___/

    */

    Each ID should have a row for time 1,2 and 3 with missing in col val1 and val2
    in case the timepoint is missing.

     INPUT                 |  RULES
                           |
     ID  TIME  VAL1  VAL2  |  ID  TIME  VAL1  VAL2
                           |
                           >   1    1     .    .    Add this because time=1 is missing
      1    2     4         |
      1    2     3    O    |
      1    2     3    A    |
      1    3     3    F    |
                           |
      2    1     3    A    |
      2    2     2    B    |
      2    2     1    L    |
                           >   2    3     .    .   Add this because time=3 is missing
                           |
                           ?   3    1     .    .   Add this because time=1 is missing
      3    2     2    B    |
      3    3     2    O    |


     Solutions

           1. SAS pure SQL
           2. SAS SQL with DOSUBL
           3. R SQL
           4. Python SQL


     /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    libname sd1 "d:/sd1";

    data sd1.have;
     input ID time val1 val2 $;
    cards4;
    1 2 4 .
    1 2 3 O
    1 2 3 A
    1 3 3 F
    2 1 3 A
    2 2 2 B
    2 2 1 L
    3 2 2 B
    3 3 2 O
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Up to 40 obs from SD1.HAVE total obs=9 19SEP2022:07:08:53                                                              */
    /*                                                                                                                        */
    /* Obs    ID    TIME    VAL1    VAL2                                                                                      */
    /*                                                                                                                        */
    /*  1      1      2       4                                                                                               */
    /*  2      1      2       3      O                                                                                        */
    /*  3      1      2       3      A                                                                                        */
    /*  4      1      3       3      F                                                                                        */
    /*  5      2      1       3      A                                                                                        */
    /*  6      2      2       2      B                                                                                        */
    /*  7      2      2       1      L                                                                                        */
    /*  8      3      2       2      B                                                                                        */
    /*  9      3      3       2      O                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/


    /*
     _                                                      _
    / |    ___  __ _ ___   _ __  _   _ _ __ ___   ___  __ _| |
    | |   / __|/ _` / __| | `_ \| | | | `__/ _ \ / __|/ _` | |
    | |_  \__ \ (_| \__ \ | |_) | |_| | | |  __/ \__ \ (_| | |
    |_(_) |___/\__,_|___/ | .__/ \__,_|_|  \___| |___/\__, |_|
                          |_|                            |_|
    */

    * create a nine row template with all combinations of 1-3 with 1-3
      and left join th template with the have dataset;

    proc sql;
     create
       table want as
     select
        l.inId                 as id
       ,coalesce(l.tym,r.time) as time
       ,r.val1
       ,r.val2
     from
       ( select
           distinct Id as inId
          ,tym
       from
           sd1.have as l full outer join
             ( select
                 distinct time as tym
               from
                 sd1.have
             ) as r
       on
          1 =1
       ) as l left join sd1.have as r
     on
                 l.inId = r.id
           and   l.tym  = r.time
    order
           by Id ,time ,val1
    ;quit;

    /*___                _    ___         _                 _     _
    |___ \     ___  __ _| |  ( _ )     __| | ___  ___ _   _| |__ | |
      __) |   / __|/ _` | |  / _ \/\  / _` |/ _ \/ __| | | | `_ \| |
     / __/ _  \__ \ (_| | | | (_>  < | (_| | (_) \__ \ |_| | |_) | |
    |_____(_) |___/\__, |_|  \___/\/  \__,_|\___/|___/\__,_|_.__/|_|
                      |_|
    */

    %macro dosubl(arg);
      %let rc=%qsysfunc(dosubl(&arg));
    %mend dosubl;

    * note dubl is executed before the SQL query so the table the cartesian
      table is available for the query;

    proc sql;
     create
       table want as
     select
        l.inId                 as id
       ,coalesce(l.tym,r.time) as time
       ,r.val1
       ,r.val2
     from
        cartesian
           %dosubl('
               data cartesian;
                 do inId= 1 to 3;
                    do tym= 1 to 3;
                      output;
                    end;
                 end;
               run;quit;
               ')  as l left join sd1.have as r
     on
                 l.inId = r.id
           and   l.tym  = r.time
    order
           by Id ,time ,val1
    ;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  Up to 40 obs WORK.WANT total obs=12 19SEP2022:07:59:37                                                                */
    /*                                                                                                                        */
    /*  Obs    ID    TIME    VAL1    VAL2                                                                                     */
    /*                                                                                                                        */
    /*    1     1      1       .                                                                                              */
    /*    2     1      2       3      A                                                                                       */
    /*    3     1      2       3      O                                                                                       */
    /*    4     1      2       4                                                                                              */
    /*    5     1      3       3      F                                                                                       */
    /*    6     2      1       3      A                                                                                       */
    /*    7     2      2       1      L                                                                                       */
    /*    8     2      2       2      B                                                                                       */
    /*    9     2      3       .                                                                                              */
    /*   10     3      1       .                                                                                              */
    /*   11     3      2       2      B                                                                                       */
    /*   12     3      3       2      O                                                                                       */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____    ____              _
    |___ /   |  _ \   ___  __ _| |
      |_ \   | |_) | / __|/ _` | |
     ___) |  |  _ <  \__ \ (_| | |
    |____(_) |_| \_\ |___/\__, |_|
                             |_|
    */

     %macro utlfkil
        (
        utlfkil
        ) / des="delete an external file";

        %local urc;

        %let urc = %sysfunc(filename(fname,%quote(&utlfkil)));

        %if &urc = 0 and %sysfunc(fexist(&fname)) %then %do;
            %let urc = %sysfunc(fdelete(&fname));
            %put xxxxxx &fname deleted xxxxxx;
        %end;
        %else %do;
            %put xxxxxx &fname not found xxxxxx;
        %end;

        %let urc = %sysfunc(filename(fname,""));

      run;

    %mend utlfkil;

    %utlfkil(d:/xpt/want.xpt);

    %utl_submit_r64("
      library(haven);
      library(SASxport);
      library(sqldf);
      have<-read_sas('d:/sd1/have.sas7bdat');
      have;
      want<-sqldf('
         select
            l.inId                 as id
           ,coalesce(l.tym,r.time) as time
           ,r.val1
           ,r.val2
         from
           ( select
               distinct Id as inId
              ,tym
           from
               have as l left join
                 ( select
                     distinct time as tym
                   from
                     have
                 ) as r
           on
              1 =1
           ) as l left join have as r
         on
                     l.inId = r.id
               and   l.tym  = r.time
        order
               by Id ,time ,val1
        ');
       want;
        write.xport(want,file='d:/xpt/want.xpt');
    ");

    libname xpt xport "d:/xpt/want.xpt";

    proc print data=xpt.want;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Up to 40 obs WORK.WANT total obs=12 19SEP2022:07:58:19                                                                 */
    /*                                                                                                                        */
    /* Obs    ID    TIME    VAL1    VAL2                                                                                      */
    /*                                                                                                                        */
    /*   1     1      1       .                                                                                               */
    /*   2     1      2       3      A                                                                                        */
    /*   3     1      2       3      O                                                                                        */
    /*   4     1      2       4                                                                                               */
    /*   5     1      3       3      F                                                                                        */
    /*   6     2      1       3      A                                                                                        */
    /*   7     2      2       1      L                                                                                        */
    /*   8     2      2       2      B                                                                                        */
    /*   9     2      3       .                                                                                               */
    /*  10     3      1       .                                                                                               */
    /*  11     3      2       2      B                                                                                        */
    /*  12     3      3       2      O                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/
    /*  _                  _   _                             _
    | || |     _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
    | || |_   | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
    |__   _|  | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
       |_|(_) | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
              |_|    |___/                                |_|
    */

    You need to do a lot of setup to use Python SQL effectively

    Windows 10 64 bit Python 64 bit 3.10 and SAS 64 bit 9.4M7 pandasql installed 12/29/2021

       This opens up Python to SAS 'proc sql' programmers.

       Process
          1. Download the load module with the sql functions
          2. Place downloaded libsqlitefunctions.dll in  c:/temp/libsqlitefunctions.dll
          3. Run Python SQL code

    Download from OneDrive (place in c:/temp/libsqlitefunctions.dll)
    You can compile from source if you do not want to download. see below.

    This is the dynamically loadable windows dll with added sqlite functions (libsqlitefunctions.dll)
    https://1drv.ms/u/s!AoqaX8I7j_icgx2BAyQY9pGup-UQ?e=wbuSYO

    If you do not want to download. Not so easy but you can create the load module from source.
    https://apimirror.com/sqlite/loadext
    Python sqlite is missing these function;

    Thi is a sample of missing functions, you will be adding these to sqlite

    rounding                      : ceil, floor, trunc;
    logarithmic                   : ln, log10, log2, log;
    arithmetic                    : pow, sqrt, mod;
    trigonometric                 : cos, sin, tan;
    hyperbolic                    : cosh, sinh, tanh;
    inverse trigonometric         : acos, asin, atan, atan2;
    inverse hyperbolic            : acosh, asinh, atanh;
    angular measures              : radians, degrees; pi.

    median(x)                     : median (50th percentile),
    percentile_25(x)              : 25th percentile,
    percentile_75(x)              : 75th percentile,
    percentile_90(x)              : 90th percentile,
    percentile_95(x)              : 95th percentile,
    percentile_99(x)              : 99th percentile,
    percentile(x, perc)           : custom percentile (perc between 0 and 100),
    stddev(x) or stddev_samp(x)   : sample standard deviation,
    stddev_pop(x)                 : population standard deviation,
    variance(x) or var_samp(x)    : sample variance,
    var_pop(x)                    : population variance.


    %macro utl_pybegin39;

    %utlfkil(c:/temp/py_pgm.py);
    %utlfkil(c:/temp/py_pgm.log);
    %utlfkil(c:/temp/example.xpt);
    filename ft15f001 "c:/temp/py_pgm.py";

    %mend utl_pybegin39;

    %macro utl_pyend39;
    run;quit;

    * EXECUTE THE PYTHON PROGRAM;
    options noxwait noxsync;
    filename rut pipe  "d:\Python310\python.exe c:/temp/py_pgm.py 2> c:/temp/py_pgm.log";
    run;quit;

    data _null_;
      file print;
      infile rut;
      input;
      put _infile_;
      putlog _infile_;
    run;quit;

    data _null_;
      infile " c:/temp/py_pgm.log";
      input;
      putlog _infile_;
    run;quit;

    %mend utl_pyend39;
    /*           _   _                                 _       _
     _ __  _   _| |_| |__   ___  _ __    ___  ___ _ __(_)_ __ | |_
    | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ __| `__| | `_ \| __|
    | |_) | |_| | |_| | | | (_) | | | | \__ \ (__| |  | | |_) | |_
    | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\___|_|  |_| .__/ \__|
    |_|    |___/                                        |_|
    */

    proc datasets lib=work kill nodetails nolist;
    run;quit;

    %utlfkil(d:/xpt/res.xpt);

    %utl_pybegin39;
    parmcards4;
    from os import path
    import pandas as pd
    import xport
    import xport.v56
    import pyreadstat
    import numpy as np
    import pandas as pd
    from pandasql import sqldf
    mysql = lambda q: sqldf(q, globals())
    from pandasql import PandaSQL
    pdsql = PandaSQL(persist=True)
    sqlite3conn = next(pdsql.conn.gen).connection.connection
    sqlite3conn.enable_load_extension(True)
    sqlite3conn.load_extension('c:/temp/libsqlitefunctions.dll')
    mysql = lambda q: sqldf(q, globals())
    have, meta = pyreadstat.read_sas7bdat("d:/sd1/have.sas7bdat")
    print(have);
    res = pdsql("""
     select
        l.inId                 as id
       ,coalesce(l.tym,r.time) as time
       ,r.val1
       ,r.val2
     from
        ( select
           distinct Id as inId
          ,tym
        from
           have as l left join
             ( select
                 distinct time as tym
               from
                 have
             ) as r
        on
          1 =1 ) as l left join have as r
     on
              l.inId = r.id
        and   l.tym  = r.time
    order
        by Id ,time ,val1
        ;""")
    print(res);
    ds = xport.Dataset(res, name='res')
    with open('d:/xpt/res.xpt', 'wb') as f:
        xport.v56.dump(ds, f)
    ;;;;
    %utl_pyend39;

    libname pyxpt xport "d:/xpt/res.xpt";

    proc contents data=pyxpt._all_;
    run;quit;

    proc print data=pyxpt.res;
    run;quit;

    data res;
       set pyxpt.res;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  Up to 40 obs from RES total obs=12 19SEP2022:08:21:00                                                                 */
    /*                                                                                                                        */
    /*  Obs    ID    TIME    VAL1    VAL2                                                                                     */
    /*                                                                                                                        */
    /*    1     1      1       .                                                                                              */
    /*    2     1      2       3      A                                                                                       */
    /*    3     1      2       3      O                                                                                       */
    /*    4     1      2       4                                                                                              */
    /*    5     1      3       3      F                                                                                       */
    /*    6     2      1       3      A                                                                                       */
    /*    7     2      2       1      L                                                                                       */
    /*    8     2      2       2      B                                                                                       */
    /*    9     2      3       .                                                                                              */
    /*   10     3      1       .                                                                                              */
    /*   11     3      2       2      B                                                                                       */
    /*   12     3      3       2      O                                                                                       */
    /*                                                                                                                        */
    /**************************************************************************************************************************/


    /*           _   _                   _
     _ __  _   _| |_| |__   ___  _ __   | | ___   __ _
    | `_ \| | | | __| `_ \ / _ \| `_ \  | |/ _ \ / _` |
    | |_) | |_| | |_| | | | (_) | | | | | | (_) | (_| |
    | .__/ \__, |\__|_| |_|\___/|_| |_| |_|\___/ \__, |
    |_|    |___/                                 |___/
    */


    3361  proc datasets lib=work kill nodetails nolist;
    NOTE: Deleting WORK.RES (memtype=DATA).
    NOTE: Deleting WORK.SASGOPT (memtype=CATALOG).
    NOTE: File WORK.SASGOPT (memtype=CATALOG) cannot be deleted because it is in use.
    NOTE: Deleting WORK.SASMACR (memtype=CATALOG).
    NOTE: File WORK.SASMACR (memtype=CATALOG) cannot be deleted because it is in use.
    3362  run;

    3362!     quit;

    NOTE: PROCEDURE DATASETS used (Total process time):
          real time           0.01 seconds
          user cpu time       0.00 seconds
          system cpu time     0.01 seconds
          memory              231.43k
          OS Memory           19960.00k
          Timestamp           09/19/2022 08:25:50 AM
          Step Count                        533  Switch Count  0


    3363  %utlfkil(d:/xpt/res.xpt);
    MLOGIC(UTLFKIL):  Beginning execution.
    MLOGIC(UTLFKIL):  Parameter UTLFKIL has value d:/xpt/res.xpt
    MLOGIC(UTLFKIL):  %LOCAL  URC
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable UTLFKIL resolves to d:/xpt/res.xpt
    SYMBOLGEN:  Macro variable URC resolves to 0
    SYMBOLGEN:  Macro variable FNAME resolves to #LN01259
    MLOGIC(UTLFKIL):  %IF condition &urc = 0 and %sysfunc(fexist(&fname)) is TRUE
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable FNAME resolves to #LN01259
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    MPRINT(UTLFKIL):   run;
    MLOGIC(UTLFKIL):  Ending execution.
    3364  %utl_pybegin39;
    MLOGIC(UTL_PYBEGIN39):  Beginning execution.
    MLOGIC(UTLFKIL):  Beginning execution.
    MLOGIC(UTLFKIL):  Parameter UTLFKIL has value c:/temp/py_pgm.py
    MLOGIC(UTLFKIL):  %LOCAL  URC
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable UTLFKIL resolves to c:/temp/py_pgm.py
    SYMBOLGEN:  Macro variable URC resolves to 0
    SYMBOLGEN:  Macro variable FNAME resolves to #LN01260
    MLOGIC(UTLFKIL):  %IF condition &urc = 0 and %sysfunc(fexist(&fname)) is TRUE
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable FNAME resolves to #LN01260
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    MPRINT(UTLFKIL):   run;
    MLOGIC(UTLFKIL):  Ending execution.
    MPRINT(UTL_PYBEGIN39):  ;
    MLOGIC(UTLFKIL):  Beginning execution.
    MLOGIC(UTLFKIL):  Parameter UTLFKIL has value c:/temp/py_pgm.log
    MLOGIC(UTLFKIL):  %LOCAL  URC
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable UTLFKIL resolves to c:/temp/py_pgm.log
    SYMBOLGEN:  Macro variable URC resolves to 0
    SYMBOLGEN:  Macro variable FNAME resolves to #LN01261
    MLOGIC(UTLFKIL):  %IF condition &urc = 0 and %sysfunc(fexist(&fname)) is TRUE
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable FNAME resolves to #LN01261
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    MPRINT(UTLFKIL):   run;
    MLOGIC(UTLFKIL):  Ending execution.
    MPRINT(UTL_PYBEGIN39):  ;
    MLOGIC(UTLFKIL):  Beginning execution.
    MLOGIC(UTLFKIL):  Parameter UTLFKIL has value c:/temp/example.xpt
    MLOGIC(UTLFKIL):  %LOCAL  URC
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable UTLFKIL resolves to c:/temp/example.xpt
    SYMBOLGEN:  Macro variable URC resolves to 0
    SYMBOLGEN:  Macro variable FNAME resolves to #LN01262
    MLOGIC(UTLFKIL):  %IF condition &urc = 0 and %sysfunc(fexist(&fname)) is FALSE
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    MPRINT(UTLFKIL):   run;
    MLOGIC(UTLFKIL):  Ending execution.
    MPRINT(UTL_PYBEGIN39):  ;
    MPRINT(UTL_PYBEGIN39):   filename ft15f001 "c:/temp/py_pgm.py";
    MLOGIC(UTL_PYBEGIN39):  Ending execution.
    3365  parmcards4;
    3412  ;;;;

    3413  %utl_pyend39;
    MLOGIC(UTL_PYEND39):  Beginning execution.
    MPRINT(UTL_PYEND39):   run;
    MPRINT(UTL_PYEND39):  quit;
    MPRINT(UTL_PYEND39):   * EXECUTE THE PYTHON PROGRAM;
    MPRINT(UTL_PYEND39):   options noxwait noxsync;
    MPRINT(UTL_PYEND39):   filename rut pipe "d:\Python310\python.exe c:/temp/py_pgm.py
    2> c:/temp/py_pgm.log";
    MPRINT(UTL_PYEND39):   run;
    MPRINT(UTL_PYEND39):  quit;
    MPRINT(UTL_PYEND39):   data _null_;
    MPRINT(UTL_PYEND39):   file print;
    MPRINT(UTL_PYEND39):   infile rut;
    MPRINT(UTL_PYEND39):   input;
    MPRINT(UTL_PYEND39):   put _infile_;
    MPRINT(UTL_PYEND39):   putlog _infile_;
    MPRINT(UTL_PYEND39):   run;

    NOTE: The infile RUT is:
          Unnamed Pipe Access Device,
          PROCESS=d:\Python310\python.exe c:/temp/py_pgm.py 2> c:/temp/py_pgm.log,
          RECFM=V,LRECL=384

        ID  TIME  VAL1 VAL2
    0  1.0   2.0   4.0
    1  1.0   2.0   3.0    O
    2  1.0   2.0   3.0    A
    3  1.0   3.0   3.0    F
    4  2.0   1.0   3.0    A
    5  2.0   2.0   2.0    B
    6  2.0   2.0   1.0    L
    7  3.0   2.0   2.0    B
    8  3.0   3.0   2.0    O

         id  time  VAL1  VAL2
    0   1.0   1.0   NaN  None
    1   1.0   2.0   3.0     A
    2   1.0   2.0   3.0     O
    3   1.0   2.0   4.0
    4   1.0   3.0   3.0     F
    5   2.0   1.0   3.0     A
    6   2.0   2.0   1.0     L
    7   2.0   2.0   2.0     B
    8   2.0   3.0   NaN  None
    9   3.0   1.0   NaN  None
    10  3.0   2.0   2.0     B
    11  3.0   3.0   2.0     O
    NOTE: 23 lines were written to file PRINT.
    NOTE: 23 records were read from the infile RUT.
          The minimum record length was 23.
          The maximum record length was 25.
    NOTE: DATA statement used (Total process time):
          real time           1.69 seconds
          user cpu time       0.14 seconds
          system cpu time     0.28 seconds
          memory              329.43k
          OS Memory           19960.00k
          Timestamp           09/19/2022 08:25:52 AM
          Step Count                        534  Switch Count  0


    MPRINT(UTL_PYEND39):  quit;
    MPRINT(UTL_PYEND39):   data _null_;
    MPRINT(UTL_PYEND39):   infile " c:/temp/py_pgm.log";
    MPRINT(UTL_PYEND39):   input;
    MPRINT(UTL_PYEND39):   putlog _infile_;
    MPRINT(UTL_PYEND39):   run;

    NOTE: The infile " c:/temp/py_pgm.log" is:
          Filename=c:\temp\py_pgm.log,
          RECFM=V,LRECL=384,File Size (bytes)=215,
          Last Modified=19Sep2022:08:25:51,
          Create Time=09Jan2022:15:17:50

    d:\Python310\lib\site-packages\xport\v56.py:661: UserWarning: Converting column dtype
    s {'VAL2': 'string'}
      warnings.warn(f'Converting column dtypes {conversions}')
    Converting column 'VAL2' from object to string
    NOTE: 3 records were read from the infile " c:/temp/py_pgm.log".
          The minimum record length was 46.
          The maximum record length was 105.
    NOTE: DATA statement used (Total process time):
          real time           0.03 seconds
          user cpu time       0.00 seconds
          system cpu time     0.01 seconds
          memory              315.31k
          OS Memory           19960.00k
          Timestamp           09/19/2022 08:25:52 AM
          Step Count                        535  Switch Count  0

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
