@echo OFF
echo.

echo  -------------------------- AutoDoc --------------------------- 
echo  ------------- Automatic Document Back-up process -------------
echo  --------------------- (C) Artiom Tsimkin ---------------------
echo.
echo  The window has to stay open / minimized
echo  in order for the software to keep working.
echo  The proccess can be stopped by pressing Ctrl+C.
echo.
echo.

set thisPath=%~dp0
set wantLog=true
set createdNewInCurrent=false

:WhatToBackup
:: which directory would you like to back up?
set /p UserInput=### Would you like to back up the current directory [Y/N]? 
echo.

:: check what to back up
if /i %UserInput%==y (
set backUpThisPath=%thisPath%
goto WhereToBackup
)

:EnterPath1
if /i %UserInput%==n (
echo  Please input the directory that you would like to back up:
echo  [ Example: C:\name\name\...\name ]
echo.
set /p backUpThisPath=
echo.

) else (
echo  ERROR: Wrong input, please try again.
echo.
goto WhatToBackup)

:: directory error
if not exist "%backUpThisPath%" (
echo  ERROR: The directory doesn't exist.
goto EnterPath1
) else (
goto WhereToBackup
)
echo.


:WhereToBackup
:: where would you like to back up?
set /p UserInput=### Would you like to back up into the current directory [Y/N]? 
echo.

:: check where to back up
if /i %UserInput%==y (
set backUpToHere=%thisPath%
goto ChooseLog
)

:EnterPath2
if /i %UserInput%==n (
echo  Please input the directory of where you would like your back up:
echo  [ Example: C:\name\name\...\name\ ]
echo  Specifing an invalid path will create the new directory in this folder.
echo  [ Do not use periods in the directory ]
echo.
set /p backUpToHere=

) else (
echo  ERROR: Wrong input, please try again.
echo.
goto WhereToBackup)

:: directory error
if "%backUpToHere%"=="" (
echo  Please enter a valid path.
goto EnterPath2
)

if not exist "%backUpToHere%" (
echo.
echo  WARNING: The directory doesn't exist.
goto NewDirectory
) else (
echo.
goto ChooseLog)


:NewDirectory
set /p UserInput=### Would you like to create this new directory [Y/N]? 
echo.

if /i %UserInput%==y (
set backUpToHere=%backUpToHere: =_%
set createdNewInCurrent=true
goto ChooseLog
)

if /i %UserInput%==n (
set backUpToHere=
goto EnterPath2

) else (
echo  ERROR: Wrong input, please try again.
echo.
goto NewDirectory)
echo.


:ChooseLog
:: would you like a log?
set /p UserInput=### Would you like a log in your back up folder [Y/N]? 
echo.

:: check user choise
if /i %UserInput%==y (
set wantLog=true
goto ChooseRepeat
)

if /i %UserInput%==n (
set wantLog=false
goto ChooseRepeat

) else (
echo  ERROR: Wrong input, please try again.
goto ChooseLog)


:ChooseRepeat
:: would you like it to repeat?
set /p UserInput=### Would you like the process to repeat automatically [Y/N]? 
echo.
set repeat=true
if /i %UserInput%==y (
echo  How often?
echo   1. every minute
echo   2. every 5 minutes
echo   3. every 10 minutes
echo   4. every 30 minutes
echo   5. every hour
echo   6. every 3 hours
echo   7. every 6 hours
echo   8. every 12 hours
echo   9. every 24 hours
echo.
) else (
goto WhenBackUp)

set /p repeatChoise=### Choose a number: 
echo.

if %repeatChoise%==1 (
set delayTime=minute
set delayCount=60
goto AutomaticBackup
)
if %repeatChoise%==2 (
set delayTime=5 minutes
set delayCount=300
goto AutomaticBackup
)
if %repeatChoise%==3 (
set delayTime=10 minutes
set delayCount=600
goto AutomaticBackup
)
if %repeatChoise%==4 (
set delayTime=30 minutes
set delayCount=1800
goto AutomaticBackup
)
if %repeatChoise%==5 (
set delayTime=hour
set delayCount=3600
goto AutomaticBackup
)
if %repeatChoise%==6 (
set delayTime=3 hours
set delayCount=10800
goto AutomaticBackup
)
if %repeatChoise%==7 (
set delayTime=6 hours
set delayCount=21600
goto AutomaticBackup
)
if %repeatChoise%==8 (
set delayTime=12 hours
set delayCount=43200
goto AutomaticBackup
)
if %repeatChoise%==9 (
set delayTime=24 hours
set delayCount=86400
goto AutomaticBackup
) else (
echo  ERROR: Wrong input, please try again.
echo.
goto ChooseRepeat)


set BackUpIndex=0
:: loop inside automatic back up
:AutomaticBackup
echo ---------------------------------------------------------------
echo.
set /A BackUpIndex=%BackUpIndex%+1
echo  ### Automatic back up is running.
echo  ### This is back up number: %BackUpIndex%
echo  ### The software is performing a back up every %delayTime%.
echo  ### The proccess can be stopped by pressing Ctrl+C.
goto BackUp


:AutomaticBackupDelay
TIMEOUT %delayCount% /NOBREAK >NUL
goto AutomaticBackup


:: user chose not to repeat
:WhenBackUp
if /i %UserInput%==n (
set /p repeatOnce=### Would you like to back up now [Y/N]? 
set repeat=false
echo.

) else (
echo  ERROR: Wrong input, please try again.
echo.
goto ChooseRepeat)

if /i %repeatOnce%==y (
echo ---------------------------------------------------------------
goto BackUp
)


:: choose when to do a 1 time back up
:EnterTime
if /i %repeatOnce%==n (
echo  Please enter when would you like to back up.
echo  Use the format:  Date,Hour:Minute:Second
echo  [ Example: 24,17:05:13]
echo  All should be written in 2 digit form.
echo  Hour format [12/24] should be the similar to
echo  what is set up on your machine.
echo  In case of a wrong input the back up will never occur.
echo.
set /p BackUpTime=
echo.
echo  Software is running. Waiting for the scheduled back up time...
echo.
goto WaitForTime

) else (
echo  ERROR: Wrong input, please try again.
echo.
goto WhenBackUp)


:: wait until chosen time comes
:WaitForTime
if %BackUpTime:~0,2% NEQ %date:~0,2% (
goto sleep)
if %BackUpTime:~3,2% NEQ %time:~0,2% (
goto sleep)
if %BackUpTime:~6,2% NEQ %time:~3,2% (
goto sleep)
if %BackUpTime:~9,2% NEQ %time:~6,2% (
goto sleep
) else (
echo ---------------------------------------------------------------
goto BackUp
)


:sleep
TIMEOUT 1 /NOBREAK >NUL
goto WaitForTime


:: do the back up
:BackUp
echo.
echo  Starting...
echo  Time of back up: %DATE% %TIME:~0,8%
echo  Using ROBOCOPY to copy your files...

::if the time is 1 digit / less than 10, adjust folder name
if %TIME:~0,2% LSS 10 (
set Mytime=0%TIME:~1,1%;%TIME:~3,2%
) else (
set Mytime=%TIME:~0,2%;%TIME:~3,2%)

:: create new folder and copy files
set newFolderName=Back-Up__%date:/=%__%Mytime%
set newFolderPath=%backUpToHere%\%newFolderName%
mkdir "%newFolderPath%"

if %wantLog%==true (
ROBOCOPY "%backUpThisPath% " "%newFolderPath%" > "BACK UP LOG.txt" /e /v /ts /eta /tee /fp /bytes /mir /R:5 /W:5 /XD %newFolderName% %backUpToHere%
) else (
ROBOCOPY "%backUpThisPath% " "%newFolderPath%" /e /NFL /NDL /NJH /NJS /nc /ns /np /mir /R:5 /W:5 /XD %newFolderName% %backUpToHere%)

echo  Done.
goto checkCopyStatus
echo.


:checkCopyStatus
if %ERRORLEVEL%==0 (
echo  STATUS: No files were copied. No failure was encountered.
echo  No files were mismatched.
echo  The files already exist in the destination directory;
echo  therefore, the copy operation was skipped.
)
if %ERRORLEVEL%==1 (
echo  STATUS: All files were copied successfully.
)
if %ERRORLEVEL%==2 (
echo  STATUS: There are some additional files in the destination directory that
echo  are not present in the source directory.
echo  No files were copied.
)
if %ERRORLEVEL%==3 (
echo  STATUS: Some files were copied. Additional files were present.
echo  No failure was encountered.
)
if %ERRORLEVEL%==5 (
echo  STATUS: Some files were copied. Some files were mismatched. 
echo  No failure was encountered.
)
if %ERRORLEVEL%==6 (
echo  STATUS: Additional files and mismatched files exist.
echo  No files were copied and no failures were encountered.
echo  This means that the files already exist in the destination directory.
)
if %ERRORLEVEL%==7 (
echo  STATUS: Files were copied, a file mismatch was present,
echo  and additional files were present.
)
if %ERRORLEVEL%==8 (
echo  STATUS: Several files did not copy.
)

echo.
echo ------------------------------
goto DeleteLeftovers


:: delete left overs
:DeleteLeftovers
if %wantLog%==true (
move "BACK UP LOG.txt" "%newFolderPath%" > leftOvers.txt
del leftOvers.txt
)


:: if repeat is on go back
if %repeat%==true (
goto AutomaticBackupDelay
) else (
goto end)


::exit
:end
echo.
echo.
pause
echo.