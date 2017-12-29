# utl_python_safe_encrypting_and_decrypting_PII_in_SAS_WPS_tables
    Python safe encrypting and decrypting of PII in SAS or WPS tables                                                       
                                                                                                                            
     WPS implementation                                                                                                     
           1. requires new utl_submit_wps64 macro if executing from SAS (beta)                                              
              github                                                                                                        
              https://github.com/rogerjdeangelis/utl_submit_wps64/blob/master/utl_submit_wps64_proc_python                  
              macro also onthe end of this post                                                                             
           2. Could not get it to work in Python 2.7 so use Python 2.5 or later.                                            
           3. Used default lengths for encryption                                                                           
                                                                                                                            
                                                                                                                            
    see                                                                                                                     
    https://goo.gl/7nTiJ7                                                                                                   
    https://github.com/rogerjdeangelis/utl_python_safe_encrypting_and_decrypting_PII_in_SAS_WPS_tables                      
                                                                                                                            
    Below are two python programs to encrypt and decrypt text.                                                              
                                                                                                                            
     1. The first inputs a SAS/WPS table then encrypts a column                                                             
        containing SSNs. Finally it outputs a SAS/WPS table with encrypted SSNs.                                            
                                                                                                                            
                                                   ENCRYPTED_SSN                                                            
                                                                                                                            
        gAAAAABaRomFB0abvmi58CsSSn_ZZbyOivjvbnPxx4Dw8-abE3j4XEQXNBUeeRRg2IhGrqFgMfA7_LmmspP30wUiPbWtYKDg6A==                
        gAAAAABaRomFu12NctBZ2gX1i095gIXUbTdfyBGXy4KvvgYQDFnrjPfflwFz4hwHNndXfY-OI6kuyIL_xHsI-E-ZjinABJwICQ==                
        gAAAAABaRomF2hDG0gX2rrdl38qqFLKRx7uouNLOyN0UiSeNYDYp3e0JQvA1AiCkJ6SMlsqGGf9t7BklNKgI0FASSU-xVuYpnQ==                
                                                                                                                            
     2, The second program inputs the SAS/WPS table with encrypted SSNs, decrypts the SSN and outputs                       
        a SAS/WPS table.                                                                                                    
                                                                                                                            
        DECRYPTED_                                                                                                          
           SSN ( randomly generated SSNs)                                                                                   
                                                                                                                            
        872916514                                                                                                           
        893965554                                                                                                           
        907470767                                                                                                           
                                                                                                                            
                                                                                                                            
                                                                                                                            
    INPUT ENCRYPT python program                                                                                            
    ==============================                                                                                          
                                                                                                                            
       SD1.HAVE total obs=3                                                                                                 
                                                                                                                            
            SSN  (randomly created)                                                                                         
                                                                                                                            
         872916514                                                                                                          
         893965554                                                                                                          
         907470767                                                                                                          
                                                                                                                            
                                                                                                                            
    INPUT DECRYPT PROGRAM  (OUTPUT of ENCRYPT program above)                                                                
    =========================================================                                                               
                                                                                                                            
         cipher_key = b'ZkvzzKVG-YVhsQzU4Ny96XrBXRunFtsd6iEgjTk-JCM=';  * need to get this from ENCRYPT output;             
                                                                        * or you can build your own;                        
                                                    ENCRYPTED_SSN                                                           
                                                                                                                            
         gAAAAABaRomFB0abvmi58CsSSn_ZZbyOivjvbnPxx4Dw8-abE3j4XEQXNBUeeRRg2IhGrqFgMfA7_LmmspP30wUiPbWtYKDg6A==               
         gAAAAABaRomFu12NctBZ2gX1i095gIXUbTdfyBGXy4KvvgYQDFnrjPfflwFz4hwHNndXfY-OI6kuyIL_xHsI-E-ZjinABJwICQ==               
         gAAAAABaRomF2hDG0gX2rrdl38qqFLKRx7uouNLOyN0UiSeNYDYp3e0JQvA1AiCkJ6SMlsqGGf9t7BklNKgI0FASSU-xVuYpnQ==               
                                                                                                                            
                                                                                                                            
    WORKING CODE (ENCRYPT)                                                                                                  
    ======================                                                                                                  
                                                                                                                            
       have = pandas.read_sas('d:/sd1/have.sas7bdat',encoding='ascii'); * load SAS table with clear text SSNs;              
       want = pd.DataFrame(columns=['encrypted_text']);                 * create empty dataframe;                           
       cipher_key = Fernet.generate_key();                              * create random (need to save this key see output;  
       print(cipher_key);                                                                                                   
       cipher = Fernet(cipher_key);                                     * iterate through rows;                             
       for row_index,row in have.iterrows():;                                                                               
            text=row.values[0];                                                                                             
            text=text.encode(encoding='utf-8');                         * create byte string;                               
            encrypted_text = cipher.encrypt(text);                      * encrypt;                                          
            decrypted_text = cipher.decrypt(encrypted_text);            * just for checking decrypt;                        
            print(decrypted_text);                                                                                          
            nxt = pd.DataFrame({'encrypted_text':[encrypted_text]});    * create one row dataframe to append;               
            want=want.append(nxt);                                      * append encrypted SSN one ata  time;               
       want.encrypted_text = want.encrypted_text.str.decode('utf-8');   * from utf-8 to ascii SAS output;                   
       endsubmit;                                                                                                           
       import  python=want data=sd1.want_encrypt;                       * send to WPS/SAS                                   
                                                                                                                            
                                                                                                                            
    WORKING CODE (DECRYPT)                                                                                                  
    ======================                                                                                                  
                                                                                                                            
       * input is output from ENCRYPT program;                                                                              
                                                                                                                            
        wantwps = pandas.read_sas('d:/sd1/wantwps.sas7bdat',encoding='ascii');  * load SAS table with clear text SSNs;      
        want = pd.DataFrame(columns=['decrypted_text']);                        * create empty dataframe;                   
        cipher_key = b'ZkvzzKVG-YVhsQzU4Ny96XrBXRunFtsd6iEgjTk-JCM=';           * paste cipher from output above;           
        cipher = Fernet(cipher_key);                                            * iterate through rows;                     
        for row_index,row in wantwps.iterrows():;                                                                           
        .    text=row.values[0];                                                * create byte strings;                      
        .    row=text.encode(encoding='utf-8');                                 * decrypt;                                  
        .    decrypted_text = cipher.decrypt(row);                                                                          
        .    nxt = pd.DataFrame({'decrypted_text':[decrypted_text]});                                                       
        .    want=want.append(nxt);                                             * append encrypted SSN one ata  time;       
        want.decrypted_text = want.decrypted_text.str.decode('utf-8');          * from utf-8 to ascii SAS output;           
        endsubmit;                                                              * send to WPS/SAS                           
        import  python=want data=sd1.want_decrypt;                                                                          
                                                                                                                            
                                                                                                                            
    OUTPUT (ENCRYPT)                                                                                                        
    ================                                                                                                        
         cipher_key = b'ZkvzzKVG-YVhsQzU4Ny96XrBXRunFtsd6iEgjTk-JCM=';  * need to get this from ENCRYPT output;             
                                                                        * or you can build your own;                        
                                                    ENCRYPTED_SSN                                                           
                                                                                                                            
         gAAAAABaRomFB0abvmi58CsSSn_ZZbyOivjvbnPxx4Dw8-abE3j4XEQXNBUeeRRg2IhGrqFgMfA7_LmmspP30wUiPbWtYKDg6A==               
         gAAAAABaRomFu12NctBZ2gX1i095gIXUbTdfyBGXy4KvvgYQDFnrjPfflwFz4hwHNndXfY-OI6kuyIL_xHsI-E-ZjinABJwICQ==               
         gAAAAABaRomF2hDG0gX2rrdl38qqFLKRx7uouNLOyN0UiSeNYDYp3e0JQvA1AiCkJ6SMlsqGGf9t7BklNKgI0FASSU-xVuYpnQ==               
                                                                                                                            
    OUTPUT (DECRYPT)                                                                                                        
    ================                                                                                                        
                                                                                                                            
       SD1.WANT_DECRYPT                                                                                                     
                                                                                                                            
            SSN                                                                                                             
                                                                                                                            
         872916514                                                                                                          
         893965554                                                                                                          
         907470767                                                                                                          
                                                                                                                            
    ADDITIONAL EXPLANATION FROM LINK                                                                                        
    =================================                                                                                       
                                                                                                                            
    Python safe encrypting and decrypting of PII in SAS or WPS tables                                                       
                                                                                                                            
    https://dzone.com/articles/an-intro-to-encryption-in-python-3                                                           
                                                                                                                            
     "First off we need to import Fernet. Next we generate a key.                                                           
     We print out the key to see what it looks like. As you can see,                                                        
     it’s a random byte string. If you want, you can try running                                                            
     the generate_key method a few times. The result will always be different.                                              
     Next we create our Fernet cipher instance using our key.                                                               
                                                                                                                            
     Now we have a cipher we can use to encrypt and decrypt our message.                                                    
     The next step is to create a message worth encrypting and then encrypt it                                              
     using the encrypt method. I went ahead and printed our                                                                 
     the encrypted text so you can see that you can no longer read the text.                                                
     To decrypt our super secret message, we just call decrypt on our                                                       
     cipher and pass it the encrypted text. The result                                                                      
     is we get a plain text byte string of our message."                                                                    
    *                _              _       _                                                                               
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _                                                                        
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |                                                                       
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |                                                                       
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|                                                                       
                                                                                                                            
    ;                                                                                                                       
                                                                                                                            
    options validvarname=upcase;                                                                                            
    libname sd1 "d:/sd1";                                                                                                   
    data sd1.have(keep=ssn);                                                                                                
      retain ssn ;                                                                                                          
      set sashelp.class(obs=3);                                                                                             
      ssn=put(1000000000*uniform(77934),z9.);                                                                               
    run;quit;                                                                                                               
                                                                                                                            
                                                                                                                            
    *                                 _                                                                                     
      ___ _ __   ___ _ __ _   _ _ __ | |_                                                                                   
     / _ \ '_ \ / __| '__| | | | '_ \| __|                                                                                  
    |  __/ | | | (__| |  | |_| | |_) | |_                                                                                   
     \___|_| |_|\___|_|   \__, | .__/ \__|                                                                                  
                          |___/|_|                                                                                          
    ;                                                                                                                       
                                                                                                                            
    %utl_submit_wps64("                                                                                                     
    options set=PYTHONHOME 'C:\Users\backup\AppData\Local\Programs\Python\Python35\';                                       
    options set=PYTHONPATH 'C:\Users\backup\AppData\Local\Programs\Python\Python35\lib\';                                   
    libname sd1 'd:/sd1';                                                                                                   
    proc python;                                                                                                            
    submit;                                                                                                                 
    import numpy as np;                                                                                                     
    import pandas as pd;                                                                                                    
    from cryptography.fernet import Fernet;                                                                                 
    from sas7bdat import SAS7BDAT;                                                                                          
    from pandas import pandas;                                                                                              
    have = pandas.read_sas('d:/sd1/have.sas7bdat',encoding='ascii');                                                        
    want = pd.DataFrame(columns=['encrypted_text']);                                                                        
    cipher_key = Fernet.generate_key();                                                                                     
    print(cipher_key);                                                                                                      
    cipher = Fernet(cipher_key);                                                                                            
    for row_index,row in have.iterrows():;                                                                                  
    .    text=row.values[0];                                                                                                
    .    text=text.encode(encoding='utf-8');                                                                                
    .    encrypted_text = cipher.encrypt(text);                                                                             
    .    decrypted_text = cipher.decrypt(encrypted_text);                                                                   
    .    print(decrypted_text);                                                                                             
    .    nxt = pd.DataFrame({'encrypted_text':[encrypted_text]});                                                           
    .    want=want.append(nxt);                                                                                             
    want.encrypted_text = want.encrypted_text.str.decode('utf-8');                                                          
    endsubmit;                                                                                                              
    import  python=want data=sd1.want_encrypt;                                                                              
    run;quit;                                                                                                               
    ");                                                                                                                     
                                                                                                                            
    /* LOG                                                                                                                  
    The WPS System                                                                                                          
                                                                                                                            
    The PYTHON Procedure                                                                                                    
                                                                                                                            
    b'jyvA_UPtk4M2BVVfwc7ur8-bKRFQOxRYd5YTG83HCI0='  * need to copy this to paste in decrtpt;                               
                                                                                                                            
    b'872916514'  * printed for checking only not used anywhere else;                                                       
    b'893965554'                                                                                                            
    b'907470767'                                                                                                            
                                                    ENCRYPTED_SSN                                                           
                                                                                                                            
         gAAAAABaRomFB0abvmi58CsSSn_ZZbyOivjvbnPxx4Dw8-abE3j4XEQXNBUeeRRg2IhGrqFgMfA7_LmmspP30wUiPbWtYKDg6A==               
         gAAAAABaRomFu12NctBZ2gX1i095gIXUbTdfyBGXy4KvvgYQDFnrjPfflwFz4hwHNndXfY-OI6kuyIL_xHsI-E-ZjinABJwICQ==               
         gAAAAABaRomF2hDG0gX2rrdl38qqFLKRx7uouNLOyN0UiSeNYDYp3e0JQvA1AiCkJ6SMlsqGGf9t7BklNKgI0FASSU-xVuYpnQ==               
    */                                                                                                                      
                                                                                                                            
                                                                                                                            
    *    _                            _                                                                                     
      __| | ___  ___ _ __ _   _ _ __ | |_                                                                                   
     / _` |/ _ \/ __| '__| | | | '_ \| __|                                                                                  
    | (_| |  __/ (__| |  | |_| | |_) | |_                                                                                   
     \__,_|\___|\___|_|   \__, | .__/ \__|                                                                                  
                          |___/|_|                                                                                          
    ;                                                                                                                       
                                                                                                                            
                                                                                                                            
    %utl_submit_wps64("                                                                                                     
    options set=PYTHONHOME 'C:\Users\backup\AppData\Local\Programs\Python\Python35\';                                       
    options set=PYTHONPATH 'C:\Users\backup\AppData\Local\Programs\Python\Python35\lib\';                                   
    libname sd1 'd:/sd1';                                                                                                   
    proc python;                                                                                                            
    submit;                                                                                                                 
    import numpy as np;                                                                                                     
    import pandas as pd;                                                                                                    
    from cryptography.fernet import Fernet;                                                                                 
    from sas7bdat import SAS7BDAT;                                                                                          
    from pandas import pandas;                                                                                              
    wantwps = pandas.read_sas('d:/sd1/want_encrypt.sas7bdat',encoding='ascii');                                             
    want = pd.DataFrame(columns=['decrypted_text']);                                                                        
    cipher_key = b'jyvA_UPtk4M2BVVfwc7ur8-bKRFQOxRYd5YTG83HCI0=';                                                           
    cipher = Fernet(cipher_key);                                                                                            
    for row_index,row in wantwps.iterrows():;                                                                               
    .    text=row.values[0];                                                                                                
    .    row=text.encode(encoding='utf-8');                                                                                 
    .    decrypted_text = cipher.decrypt(row);                                                                              
    .    nxt = pd.DataFrame({'decrypted_text':[decrypted_text]});                                                           
    .    want=want.append(nxt);                                                                                             
    want.decrypted_text = want.decrypted_text.str.decode('utf-8');                                                          
    print(want);                                                                                                            
    endsubmit;                                                                                                              
    import  python=want data=sd1.want_decrypt;                                                                              
    run;quit;                                                                                                               
    ");                                                                                                                     
                                                                                                                            
    /*                                                                                                                      
                                                                                                                            
      The WPS System                                                                                                        
                                                                                                                            
      The PYTHON Procedure                                                                                                  
                                                                                                                            
        decrypted_text                                                                                                      
      0      872916514                                                                                                      
      0      893965554                                                                                                      
      0      907470767                                                                                                      
                                                                                                                            
                                                                                                                            
      SD1.WANT_DECRYPTtotal obs=3                                                                                           
                                                                                                                            
       DECRYPTED_                                                                                                           
          TEXT                                                                                                              
                                                                                                                            
       872916514                                                                                                            
       893965554                                                                                                            
       907470767                                                                                                            
    */                                                                                                                      
                                                                                                                            
    *          _               _ _                         __   _  _                                                        
     ___ _   _| |__  _ __ ___ (_) |_  __      ___ __  ___ / /_ | || |                                                       
    / __| | | | '_ \| '_ ` _ \| | __| \ \ /\ / / '_ \/ __| '_ \| || |_                                                      
    \__ \ |_| | |_) | | | | | | | |_   \ V  V /| |_) \__ \ (_) |__   _|                                                     
    |___/\__,_|_.__/|_| |_| |_|_|\__|___\_/\_/ | .__/|___/\___/   |_|                                                       
                                   |_____|     |_|                                                                          
    ;                                                                                                                       
                                                                                                                            
    %macro utl_submit_wps64(pgmx,resolve=Y)/des="submiit a single quoted sas program to wps";                               
      * write the program to a temporary file;                                                                              
                                                                                                                            
      %utlfkil(%sysfunc(pathname(work))/wps_pgmtmp.wps);                                                                    
      %utlfkil(%sysfunc(pathname(work))/wps_pgm.wps);                                                                       
      %utlfkil(%sysfunc(pathname(work))/wps_pgm001.wps);                                                                    
      %utlfkil(wps_pgm.lst);                                                                                                
                                                                                                                            
      filename wps_pgm "%sysfunc(pathname(work))/wps_pgmtmp.wps" lrecl=32756 recfm=v;                                       
      data _null_;                                                                                                          
        length pgm  $32756 cmd $32756;                                                                                      
        file wps_pgm ;                                                                                                      
        %if %upcase(%substr(&resolve,1,1))=Y %then %do;                                                                     
           pgm=resolve(&pgmx);                                                                                              
        %end;                                                                                                               
        %else %do;                                                                                                          
          pgm=&pgmx;                                                                                                        
        %end;                                                                                                               
        semi=countc(pgm,';');                                                                                               
          do idx=1 to semi;                                                                                                 
            cmd=cats(scan(pgm,idx,';'),';');                                                                                
            len=length(strip(cmd));                                                                                         
            put cmd $varying32756. len;                                                                                     
            putlog cmd $varying32756. len;                                                                                  
          end;                                                                                                              
      run;                                                                                                                  
                                                                                                                            
      filename wps_001 "%sysfunc(pathname(work))/wps_pgm001.wps" lrecl=255 recfm=v ;                                        
      data _null_ ;                                                                                                         
        length textin $ 32767 textout $ 255 ;                                                                               
        file wps_001;                                                                                                       
        infile "%sysfunc(pathname(work))/wps_pgmtmp.wps" lrecl=32767 truncover;                                             
        format textin $char32767.;                                                                                          
        input textin $char32767.;                                                                                           
        putlog _infile_;                                                                                                    
        if lengthn( textin ) <= 255 then put textin ;                                                                       
        else do while( lengthn( textin ) > 255 ) ;                                                                          
           textout = reverse( substr( textin, 1, 255 )) ;                                                                   
           ndx = index( textout, ' ' ) ;                                                                                    
           if ndx then do ;                                                                                                 
              textout = reverse( substr( textout, ndx + 1 )) ;                                                              
              put textout $char255. ;                                                                                       
              textin = substr( textin, 255 - ndx + 1 ) ;                                                                    
        end ;                                                                                                               
        else do;                                                                                                            
          textout = substr(textin,1,255);                                                                                   
          put textout $char255. ;                                                                                           
          textin = substr(textin,255+1);                                                                                    
        end;                                                                                                                
        if lengthn( textin ) le 255 then put textin $char255. ;                                                             
        end ;                                                                                                               
      run ;                                                                                                                 
                                                                                                                            
      %put ****** file %sysfunc(pathname(work))/wps_pgm.wps ****;                                                           
                                                                                                                            
      filename wps_fin "%sysfunc(pathname(work))/wps_pgm.wps" lrecl=255 recfm=v ;                                           
      data _null_;                                                                                                          
          retain switch 0;                                                                                                  
          infile wps_001;                                                                                                   
          input;                                                                                                            
          file wps_fin;                                                                                                     
          if substr(_infile_,1,1) = '.' then  _infile_= substr(left(_infile_),2);                                           
          select;                                                                                                           
             when(left(upcase(_infile_))=:'SUBMIT;')     switch=1;                                                          
             when(left(upcase(_infile_))=:'ENDSUBMIT;')  switch=0;                                                          
             otherwise;                                                                                                     
          end;                                                                                                              
          if lag(switch)=1 then  _infile_=compress(_infile_,';');                                                           
          if left(upcase(_infile_))= 'ENDSUBMIT' then _infile_=cats(_infile_,';');                                          
          put _infile_;                                                                                                     
          putlog _infile_;                                                                                                  
      run;quit;                                                                                                             
                                                                                                                            
      %let _loc=%sysfunc(pathname(wps_fin));                                                                                
      %let _w=%sysfunc(compbl(C:/Progra~1/worldp~1/bin/wps.exe -autoexec c:\oto\Tut_Otowps.sas -config c:\cfg\wps.cfg));    
      %put &_loc;                                                                                                           
                                                                                                                            
      filename rut pipe "&_w -sysin &_loc";                                                                                 
      data _null_;                                                                                                          
        file print;                                                                                                         
        infile rut;                                                                                                         
        input;                                                                                                              
        put _infile_;                                                                                                       
        putlog _infile_;                                                                                                    
      run;                                                                                                                  
                                                                                                                            
                                                                                                                            
      filename rut clear;                                                                                                   
      filename wps_pgm clear;                                                                                               
      data _null_;                                                                                                          
        infile "wps_pgm.lst";                                                                                               
        input;                                                                                                              
        putlog _infile_;                                                                                                    
      run;quit;                                                                                                             
                                                                                                                            
                                                                                                                            
    %mend utl_submit_wps64;                                                                                                 
                                                                                                                            
                                                                                                                            
