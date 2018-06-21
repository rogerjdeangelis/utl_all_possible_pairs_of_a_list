# utl_all_possible_pairs_of_a_list
All possible pairs of a list. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    All possible pairs of a list

    github
    https://github.com/rogerjdeangelis/utl_all_possible_pairs_of_a_list

    SAS Forum
    https://communities.sas.com/t5/Base-SAS-Programming/Obtain-all-possible-pairs-from-a-list/m-p/470463


     Six  Solutions

       1. SQL
       2. Datastep array (I made some changes)
       3. Lexcomb (more general solution. Put code in one datastep)
       4. WPS/ProcR or IML/R
       5. proc format (combined into one datastep elimnated sql code)
       6. hash (needed to add a sequence variable?)

      Contributors

       1. https://communities.sas.com/t5/user/viewprofilepage/user-id/13884
       2. https://communities.sas.com/t5/user/viewprofilepage/user-id/18408
       3. https://communities.sas.com/t5/user/viewprofilepage/user-id/13879
       4. https://github.com/rogerjdeangelis/utl_all_possible_pairs_of_a_list
       5. https://communities.sas.com/t5/user/viewprofilepage/user-id/138205
       6. https://communities.sas.com/t5/user/viewprofilepage/user-id/138205

    INPUT
    =====

      Up to 40 obs SD1.HAVE total obs=5

      Obs    ELEMENT

       1        A
       2        B
       3        C
       4        D
       5        E


    PROCESS
    =======

    1. SQL

       proc sql;
          create table want as
          select distinct a.element,b.element as elementb
          from sd1.have as a, sd1.have as b
          where a.element<b.element;
       quit;


    2. Datastep array

       data want;

        * get meta data;
        if _n_=0 then do;
           %let rc=%sysfunc(dosubl('
             proc sql;
               select count(*) into :lenLst trimmed from sd1.have
             ;quit;
           '));
        array x{&lenLst} $ 32 _temporary_;
        end;

        set sd1.have end=last nobs=nobs;

        x{_n_}=element;

        if last then do;
          do i=1 to nobs-1;
            var1=x{i};
            do j=i+1 to nobs;
              var2=x{j};output;
              end;
          end;
        end;

        keep var1 var2;
       run;


    3. Lexcomb ( more general solution)

       %symdel n k ncomb / nowarn;
       data want;

        * get meta data - compile time info;
        if _n_=0 then do;
           %let rc=%sysfunc(dosubl('
               %let n=5;   /* total number of items */
               %let k=2;   /* number of items per row */
               %let ncomb=%sysfunc(comb(&n,&k));
               proc sql;
                 select count(*) into :lenLst from sd1.have
               ;quit;
           '));
           array x{&lenLst} $ 32 _temporary_;
           array c[&k] $3 c1-c&k;
           array i[&k];

        end;

        set sd1.have end=last nobs=nobs;
        x{_n_}=element;

        if last then do;
          do j=1 to &ncomb;
             rc=lexcombi(&n, &k, of i[*]);
             do h=1 to &k;
                c[h]=x[i[h]];
             end;
             keep c1 c2;
             output;
             put @4 j= @10 'i= ' i[*] +3 'c= ' c[*] +3 rc=;
          end;
        end;

       run;quit;


    4. WPS/ProcR or IML/R (Working code)

       want<-t(combn(have$ELEMENT, 2));
       full solution on end

    5. proc format

       %symdel m;
       data want;

        * get meta data - compile time info;
        if _n_=0 then do;
           %let rc=%sysfunc(dosubl('

              data fmtdataset;
               set sd1.have end=dne;
               seq=_n_;
               retain fmtname "fmtcode" type "n";
               rename seq = start;
               label =element;
               if dne then call symputx("m",seq);
              run;quit;

              proc format cntlin=fmtdataset;
              run;quit;

           '));
        end;

        do i=1 to &m-1;
          do j=i+1 to &m;
             elementa=put(i,fmtcode.);
             elementb=put(j,fmtcode.);
            keep elementa elementb;
            output;
          end;
        end;

    run;quit;

    5. HASH

        %symdel m/nowarn;
        data want;
         if _n_=0 then do;
            %let rc=%sysfunc(dosubl('
              proc sql;
              select max(qt) into :m  trimmed
              from have;
              quit;
              data vue / view=vue;
                set sd1.have;
                seq=_n_;
              run;quit;
            '));
          end;

        if _n_=1 then do;
        if 0 then set vue;
          dcl hash H (dataset:'vue') ;
           h.definekey  ("seq") ;
           h.definedata ("element") ;
           h.definedone () ;
        end;

        do i=1 to &m-1;
          do j=i+1 to &m;
              rc=h.find(key:i);
              elementa=element;
              rc=h.find(key:j);
              elementb=element;
              output;
          end;
        end;

        keep elementa elementb;
        run;quit;

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;


    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
       input element$;
    cards4;
    A
    B
    C
    D
    E
    ;;;;
    run;quit;

    *                                            ____
    __      ___ __  ___   _ __  _ __ ___   ___  |  _ \
    \ \ /\ / / '_ \/ __| | '_ \| '__/ _ \ / __| | |_) |
     \ V  V /| |_) \__ \ | |_) | | | (_) | (__  |  _ <
      \_/\_/ | .__/|___/ | .__/|_|  \___/ \___| |_| \_\
             |_|         |_|
    ;

    see process for SAS solutions;

    %utl_submit_wps64('
    libname sd1 "d:/sd1";
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    libname wrk  sas7bdat "%sysfunc(pathname(work))";
    libname hlp  sas7bdat "C:\Progra~1\SASHome\SASFoundation\9.4\core\sashelp";
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    library(haven);
    have<-read_sas("d:/sd1/have.sas7bdat");
    head(have);
    want<-t(combn(have$ELEMENT, 2));
    endsubmit;
    import r=want data=wrk.want;
    run;quit;
    ');
