# SSISLoadSplittingDemo
Practical use of Conditional split in SSIS to speed up your Sequential data load (Splitting the load)

Here is a practical example of how to speed up your sequential processing in SSIS:



1st thing is 1st:

Build you initial Sequential for each loop that processes all your source tables or files correctly.
![](http://4.bp.blogspot.com/-4xCE5SS6NFM/UkvphAtaUyI/AAAAAAAAGP8/fHdG8M98wnI/s1600/SecuencialFEFLoop.PNG)

Then Create and test your data flow task:
![](http://4.bp.blogspot.com/-N9T6yT1MOs4/Ukvp0kZoSMI/AAAAAAAAGQE/RoK6UJkxAlQ/s320/SecuencialDFT.PNG)

When this is done and tested create a copy of you package and start by building the following items:

1:Add a Data Flow Task to handle your list of files:
![](http://1.bp.blogspot.com/-1tuq6d-A3dc/UkvtmdhPnxI/AAAAAAAAGQQ/RljtMZaiwhg/s1600/BuildAListDFTCF.PNG)



2: In the Task add a flat file Source(1) (if your reading Files, and a SQL Server source if your reading Tables)
that reads the list of file names together with a row number(  in SQL this would be a select from sys objects using a Row_Number() function for flat files you would use power-shell or your favorite scripting tool to generate the file list)
![](http://2.bp.blogspot.com/-P6tclxnb_jM/UkvvErkZ2tI/AAAAAAAAGQc/gTvI5Vjppas/s320/BuildAListDFTDF.png)
Then Add a Derived Column transform(2) that creates 2 additional columns
![](http://3.bp.blogspot.com/-kRtWzxsLXJQ/UkvvznaqLcI/AAAAAAAAGQk/xSCnVhzM0OY/s320/BuildAListDFTDF_DCT.PNG)

The ProcessID is the modulus of the sequential numbering called index in my solution, this means that you will get an even spread from 0 to 3 across your index range. 
The second column "FullFileName" is a concatenation of a folder location and the "Filename" column from my list. I will use that as my connection string in the steps to come.


After this we create a Conditional Split transform(3):
![](http://4.bp.blogspot.com/-zdCv8vmPHOU/UkvxI0ElqAI/AAAAAAAAGQw/2F4otHhQCww/s320/BuildAListDFTDF_CSplit.PNG)
This evaluates the ProcessID and based on its value assigns an output called here P1, P2, P3 and P4.


Next is to insert the data into four record-set destinations (4,5,6,7)

They will be called from the control flow in four separate for each loops.


Now comes the fun part:

We now add four For Each Loop Container based on the ADO Collection:
Each ADO Loop is asigned to its induvidual Object Variable (As per the record-set destinations above)
![](http://4.bp.blogspot.com/-Sq21ENAeQjI/Ukv3-NkrXMI/AAAAAAAAGRA/w9jR39vJI1Y/s320/BuildAListFullCF.PNG)

Now take your previous DFT from the sequencial package and place it in each ADO Loop (Remember to change the variable mappings and add the additional flat file Data connections with the apropriate connection string expressions.



Once this is done then viola you are done...

here is the performance comparison I got:
![](http://1.bp.blogspot.com/-sYGS1BiW_GQ/Ukv5SumzeMI/AAAAAAAAGRI/OlxWMh0lDcQ/s1600/performance.PNG)

So The Sequential package executed and processed 99 files with 790 records in each file in 1 minute and 4 seconds and the parallel package ran in 23 seconds with the same load.

Now one thing to remember is that the packages ran at the same time and against the same data source and destination, so obviously there was locks and IO lag as a result, but you can see that there is definitely a performance improvement.




