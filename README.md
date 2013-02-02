eudora-mbox-analysis
====================

Collecting some old code from AQuA Mashup two years ago: https://github.com/openplanets/AQuA

For licensing conditions and everything else, please see the AQuA page. 

# AQuA: Script Methodology

#### Python script using the standard Python mbox library can be used to count individual mail messages and export them. An additional custom script was written to analyse the contents of TOC data unique to Eudora mail packages.

Step 1: Obtain count and summary from Eudora TOC file 
Step 2: Obtain count and summary from Eudora Mbox. Extract emails 
Step 3: Extract emails that potentially have attachments 

We concentrated on the .mbox file format, although we believe that the methods employed could also be used for other mailbox formats 

Message Count: 

The problem focussed initially on counting the number of emails in an mbox quickly, easily, and accurately. We analysed the internal bit stream of .mbox file, using visual checking in hex editor, and made reference to the mbox rfc RFC4155. On identifying recurring start sequences for each message we were able to develop a method of counting, based on the number of times that sequence appeared in the file. 

The count returned did not match expected number of email messages which were present when mbox file was opened in Eudora. We also discovered that the Eudora count correlated with a proprietary file with the extension TOC (table of contents). This can be interpreted as a form of index used by Eudora. To verify the count using existing well established tools further testing was undertaken to verify the count. The Mbox file was opened in Thunderbird, Eudora OSE, Eudora Light 3.0.3 (where the source file came from), separate email analysis script: http://www.stanford.edu/~pgbovine/mbox-analysis.htm - all resulted in same count and confirmed the Eudora TOC did not align with the 'actual' contents of the mbox. 

Testing showed that in certain cases one could delete an email from Eudora (i.e. send it to the trash), empty the trash and the user may consider it deleted. It would seem the Eudora index was updated and the mbox file was left untouched. As such, the remainder of the solution we have attempted to create is to provide tools to help further this investigation and allow the content owner to do further tests on other mbox files: 

The solution is split into three scripts, each output a separate summary.txt file describing the information found during each analysis: 

TOC Analysis (Eudora Table of Contents) 
- Requires a Eudora TOC file 
- To run: python eudora_toc_analysis.py yourTOC.toc 

Byte level analysis of TOC file showed that each TOC has a header 72 bytes in length and footer 32 bytes in length. Each entry is 218 bytes so we are able to generate a count of 'expected' emails by subtracting the sum of the header and footer and dividing the size of the file by 218. 
Other useful bytes include the name of the TOC file 8 bytes in. 

Each 218 byte entry into the table begins with 50 unknown bytes. Analysis on these bytes seemed to show the 48th byte relates to whether an email summary should be flagged as having an attachment. If the most significant bit of the byte is set then the email is shown as having an attachment. The remainer of the 218 bytes can be separated into the summary information about the email. For example, 32 bytes = timestamp; 64 bytes = sender; 64 bytes = subject. 8 bytes remain at the end. 

The script can be dissected to see this in further detail. The script will output a table of information stating the following: 

| email no. | timestamp | sender | subject | 

At the foot of this table is further summary information showing the mbox name, size and emails that eudora believes to have attachments. 

Mbox Summary and Email Extractor 
- To run: python eudora_mbox_summary_extractor.py yourMBOX.mbx 

This script uses the Python standard library for handling mbox: [http://docs.python.org/library/mailbox.html#mbox 
] 
We simply use this functionality to count all emails, output payload and header information for each individual email to a separate folder and provide a summary table of information showing: 

| email no. | timestamp | from | subject | 

This can be used to correlate actual emails with the listing found in a TOC file. The count is found at the bottom of the summary table in the summary file. 

Mbox Attachment Analysis 
- Requires Eudora Mbox and Eudora 'Attach' folder 
- To run: python eudora_attachment_analysis.py yourMBOX.mbx 

Baed on the existence of the string 'Attachment Converted:' within an email payload to indicate the existance of an attachment in Eudora and the non-existance of >Attachment Converted we output all the emails with suspected attachments into a separate folder for further analysis by the user and provide a summary table showing: 

| email no. | attachment no. | filename | exists in folder | full path | 

The 'exists in folder' is a column output by asking Python to query the filepath extracted from the email to see if the file can be found and does exist. This can be used as a preliminary stage before calling DROID on the command line via Python and running the file through DROID to get a better identification. 

Summary at the bottom of the summary table also shows the estimated number of attachments, number of attachments not found and the number of attachments found. 

Overview of solution 

Despite not providing a discrete solution for Eudora mail boxes we have demonstrated that we do have libraries available which will help tool developers work with this type of data in future. Python was simply the first language selected because of knowledge of how to script using it. The code is straightforward and self-documenting and it should be easy for other scripters to see how to extend it to pull more information out of the mailboxes we have analysed and manipulate it in different ways. For example, we only access a small number of the headers available in the collection emails, this information might be useful in further analyses. The solution might provide a foundation on which to build. 

How do you solve a problem like attachments? 

The biggest problem with the solution is not accessing the emails and extracting them, this is relatively trivial. The solution doesn't manage to extract attachments correctly. Eudora has shown to adopt multiple techniques to handle attachments. First there are attachments that Eudora explicitly handles. These are signalled by the string 'Attachment Converted:' and these attachments are found located in a folder external to the application. This is incredibly useful for getting strong identification results from DROID as one can point DROID at that folder and run an analysis on it - just as long as we can also tie an attachment to a specific email. 
The second problem with attachments are those that are not explicitly dealt with by Eudora but are encoded in the email. In the process of this work we discovered emails encoded in the payload as BASE64 integers or even plain text, this means attachments such as those using the RTF format are encoded in plain text with all the tags open for everyone to see. This means the problem is not simply one of detecting different encodings in a single file but something far more complicated it seems. 

Overall 

The solution does not address attachments properly. Having not worked with email in this detail before along with the revelation of the discrepancy between the Eudora index (TOC) and the mbox file the discovery about how attachments were handled was also profound and another detail archivists should take care to understand when dealing with email. While not a satisfying all encompassing solution I hope we have demonstrated tools we can use to help move this research forward and I believe there are immediate lines of investigation which we should follow to put archivists in a better position for dealing with email: 

- Find tools to identify multiple encodings in a single file / text stream 

- Understand how and when content-type should be set in an email and how often this is the case (the pattern /seemed/ to be inconsistent in the mboxes we looked at) 

- Testing and refinement of the scripts above leading to extending them to... 

- ...handle the external attachments more accurately and accurately tie the email to the attachment 

- Set up a workflow either in python or some other mechanism to tie DROID CLI into this work to provide an identification of the external attachment found and then eventually the   attachments found as part of the payload in emails. 
