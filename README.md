# utl-extract-row-corresponding-to-minimum-value-of-a-variable-by-group
Extract row corresponding to minimum value of a variable by group 
    %let pgm=utl-extract-row-corresponding-to-minimum-value-of-a-variable-by-group;

    Extract row corresponding to minimum value of a variable by group

    github
    https://tinyurl.com/4dbawf67
    https://github.com/rogerjdeangelis/utl-extract-row-corresponding-to-minimum-value-of-a-variable-by-group

    I wish to
       (1) group data by one variable (State),
       (2) within each group find the row of minimum value of another variable (Employees),
       (3) extract the entire row.

     SOLUTIONS

        1, SAS SQL
        2. WPS SQL
        3. WPS proc r native
        4. WPS PROC R SQL
        5. Python SQL

    SOAPBOX ON
     Speed may not be that critical for then smaller clinical datasets, especially phase 1 and phase 2.
     I find SQL and the SAS dtastep to be more self documenting
     Minimally you can use sql to minimize klingon python and R code.
    SOAPBOX OFF

    stackoverflow
    https://tinyurl.com/mtz39h9w
    https://stackoverflow.com/questions/24070714/extract-row-corresponding-to-minimum-value-of-a-variable-by-group

    options validvarname=upcase;
    data have;informat
    STATE $2.
    COMPANY $1.
    EMPLOYEES 8.
    ;input
    STATE COMPANY EMPLOYEES;
    cards4;
    AK A 82
    AK B 104
    AK C 37
    AK D 24
    RI E 19
    RI F 118
    RI G 88
    RI H 42
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                        |                                                               */
    /*  Up to 40 obs from HAVE total obs=8 03JUN2023:08:46:28 | RULES SELECT MIN EMPLOYEES ROW                                */
    /*                                                        |                                                               */
    /*  Obs    STATE    COMPANY    EMPLOYEES                  | STATE    COMPANY    EMPLOYEES                                 */
    /*                                                        |                                                               */
    /*   1      AK         A           82                     |                                                               */
    /*   2      AK         B          104                     |                                                               */
    /*   3      AK         C           37                     |                                                               */
    /*   4      AK         D           24  ** min employees   |  AK         D           24                                    */
    /*                                                        |                                                               */
    /*   5      RI         E           19  ** min employees   |  RI         E           19                                    */
    /*   6      RI         F          118                     |                                                               */
    /*   7      RI         G           88                     |                                                               */
    /*   8      RI         H           42                     |                                                               */
    /*                                                        |                                                               */
    /**************************************************************************************************************************/

    /*                               _
    / |    ___  __ _ ___   ___  __ _| |
    | |   / __|/ _` / __| / __|/ _` | |
    | |_  \__ \ (_| \__ \ \__ \ (_| | |
    |_(_) |___/\__,_|___/ |___/\__, |_|
                                  |_|
    */
    proc datasets lib=work nolist nodetails ;
     delete want;
    run;quit;

    proc sql;
      create
        table want as
      select
        state
       ,company
       ,employees
       ,min(employees) as employees_min
    from
       have
    group
       by state
    having
       min(employees) = employees
    ;quit;

    proc datasets lib=work nolist nodetails ;
     delete want;
    run;quit;
    /*___                                     _
    |___ \    __      ___ __  ___   ___  __ _| |
      __) |   \ \ /\ / / `_ \/ __| / __|/ _` | |
     / __/ _   \ V  V /| |_) \__ \ \__ \ (_| | |
    |_____(_)   \_/\_/ | .__/|___/ |___/\__, |_|
                       |_|                 |_|
    */
    proc datasets lib=work nolist nodetails ;
     delete want;
    run;quit;

    %let _pth=%sysfunc(pathname(work));

    %utl_submit_wps64("

    libname wrk '&_pth';

     proc print data=wrk.have;
     run;quit;

     options validvarname=any;
     proc sql;
       create
         table wrk.want as
       select
         state
        ,company
        ,employees
        ,min(employees) as employees_min
     from
        wrk.have
     group
        by state
     having
        min(employees) = employees
     ;quit;

     proc print data=wrk.want;
     run;quit;
    ");

    proc print data=want;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* Obs    STATE    COMPANY    EMPLOYEES    employees_min                                                                  */
    /*                                                                                                                        */
    /*  1      AK         D           24             24                                                                       */
    /*  2      RI         E           19             19                                                                       */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____                                                                  _   _
    |___ /  __      ___ __  ___   _ __  _ __ ___   ___   _ __   _ __   __ _| |_(_)_   _____
      |_ \  \ \ /\ / / `_ \/ __| | `_ \| `__/ _ \ / __| | `__| | `_ \ / _` | __| \ \ / / _ \
     ___) |  \ V  V /| |_) \__ \ | |_) | | | (_) | (__  | |    | | | | (_| | |_| |\ V /  __/
    |____(_)  \_/\_/ | .__/|___/ | .__/|_|  \___/ \___| |_|    |_| |_|\__,_|\__|_| \_/ \___|
                     |_|         |_|
    */
    proc datasets lib=work nolist nodetails ;
     delete want;
    run;quit;

    %let _pth=%sysfunc(pathname(work));

    %utl_submit_wps64('
    libname wrk "&_pth";
    proc r;
    export data=wrk.have r=have;
    submit;
    library(data.table);
    have=setDT(have);
    want<-have[have[ , .I[which.min(EMPLOYEES)], by = STATE]$V1];
    str(want);
    want;
    endsubmit;
    import data=wrk.want r=want;
    ');

    proc print data=want;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Up to 40 obs from WANT total obs=2 03JUN2023:09:40:40                                                                  */
    /*                                                                                                                        */
    /* Obs    STATE    COMPANY    EMPLOYEES                                                                                   */
    /*                                                                                                                        */
    /*  1      AK         D           24                                                                                      */
    /*  2      RI         E           19                                                                                      */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*  _                                                                  _
    | || |   __      ___ __  ___   _ __  _ __ ___   ___   _ __   ___  __ _| |
    | || |_  \ \ /\ / / `_ \/ __| | `_ \| `__/ _ \ / __| | `__| / __|/ _` | |
    |__   _|  \ V  V /| |_) \__ \ | |_) | | | (_) | (__  | |    \__ \ (_| | |
       |_|(_)  \_/\_/ | .__/|___/ | .__/|_|  \___/ \___| |_|    |___/\__, |_|
                      |_|         |_|                                   |_|
    */

    proc datasets lib=work nolist nodetails ;
     delete want;
    run;quit;

    %utl_submit_wps64('
    libname wrk "&_pth";
    proc r;
    export data=wrk.have r=have;
    submit;
    library(sqldf);
    want<-sqldf("
      select
         state
        ,company
        ,employees
        ,min(employees) as employees_min
     from
        have
     group
        by state
     having
        min(employees) = employees
    ");
    want;
    endsubmit;
    import data=wrk.want r=want;
    ');

    proc print data=want;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*   STATE COMPANY EMPLOYEES employees_min                                                                                */
    /*                                                                                                                        */
    /* 1    AK       D        24            24                                                                                */
    /* 2    RI       E        19            19                                                                                */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                 _   _                             _
    | ___|    _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
    |___ \   | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
     ___) |  | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    |____(_) | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
             |_|    |___/                                |_|
    */

    libname sd1 "d:/sd1";

    options validvarname=upcase;
    data sd1.have;informat
    STATE $2.
    COMPANY $1.
    EMPLOYEES 8.
    ;input
    STATE COMPANY EMPLOYEES;
    cards4;
    AK A 82
    AK B 104
    AK C 37
    AK D 24
    RI E 19
    RI F 118
    RI G 88
    RI H 42
    ;;;;
    run;quit;

    proc datasets lib=work kill nodetails nolist;
    run;quit;

    %utlfkil(d:/xpt/res.xpt);

    %utl_pybegin;
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
         state
        ,company
        ,employees
        ,min(employees) as min_employees
     from
        have
     group
        by state
     having
        min(employees) = employees
    """)
    print(res);
    lbl = list(res.columns.values)
    res  = res.rename(columns={k: k.upper()[:8] for k in res})
    print(lbl);
    ds = xport.Dataset(res, name='want')
    idx=-1
    for k, v in ds.items():
        idx=idx+1
        print(idx)
        v.label = lbl[idx]
    print(ds)
    with open('d:/xpt/res.xpt', 'wb') as f:
        xport.v56.dump(ds, f)
    ;;;;
    %utl_pyend;

    libname xpt xport "d:/xpt/res.xpt";
    ods output variables=namLbl;
    proc contents data=xpt._all_;
    run;quit;

    %array(_var _lbl,data=namLbl,var=variable label);

    %put &_var1;
    %put &_varn;

    %put &_lbl1;
    %put &_lbln;

    proc print data=xpt.want;
    run;quit;

    data want_r_long_names; /*---- cannot use name want because a view named want exists ----*/
      label
         %do_over(_var _lbl,phrase=%str(?_lbl = "?_var"))
      ;
      %utl_rens(xpt.want) ;
      set want;
    run;quit;

    /*---- if you want the generated code ----*/
    %utlnopts;
    data _null_;
    %do_over(_var _lbl,phrase=%str(put "?_lbl = '?_var'";))
    ;run;quit;

    proc print data=want;
    run;quit;

    %utlopts;

    /*---- STATE as STATE            ----*/
    /*---- COMPANY as COMPANY        ----*/
    /*---- EMPLOYEE as EMPLOYEES     ----*/
    /*---- MIN_EMPL as min_employees ----*/

    %arraydelete(_var);
    %arraydelete(_lbl);

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*   NOTE THE LONG VARIABLE NAME FROM THE DSD V5 EXPORT (SAS RENAMED THE VARIABLE USING THE V5 LABEL                      */
    /*                                                                                                                        */
    /*                                               MIN_                                                                     */
    /*    Obs    STATE    COMPANY    EMPLOYEES    EMPLOYEES                                                                   */
    /*                                                                                                                        */
    /*      1     AK         D           24           24                                                                      */
    /*      2     RI         E           19           19                                                                      */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
