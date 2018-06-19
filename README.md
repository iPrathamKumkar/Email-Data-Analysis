# Email Data Analysis

A series of applications to retrieve, process, and visualize email data from a mailing list

## Usage

The first step is to spider the email repository.
The base url is hard-coded in the **gmane.py** file and is hard-coded to the Sakai developer list.
You can spider another repository by changing that base url.
Make sure to delete the **content.sqlite** file if you switch the base url.
The gmane.py file operates as a spider, in that it runs slowly and retrieves one mail message per second.
It stores all of its data in a database and can be interrupted and re-started as often as needed.
It may take many hours to pull all the data down and so, you may need to restart several times.
Here is a run of gmane.py getting the first five messages of the Sakai developer list:

```
python gmane.py

Output:
How many messages to retrieve? 10
http://mbox.dr-chuck.net/sakai.devel/1/2 2662
    ggolden@umich.edu 2005-12-08T23:34:30-06:00 call for participation: developers documentation
http://mbox.dr-chuck.net/sakai.devel/2/3 2434
    csev@umich.edu 2005-12-09T00:58:01-05:00 report from the austin conference:  sakai developers break into song
http://mbox.dr-chuck.net/sakai.devel/3/4 3055
    kevin.carpenter@rsmart.com 2005-12-09T09:01:49-07:00 cas and sakai 1.5
http://mbox.dr-chuck.net/sakai.devel/4/5 11721
    michael.feldstein@suny.edu 2005-12-09T09:43:12-05:00 re: lms/vle rants/comments
http://mbox.dr-chuck.net/sakai.devel/5/6 9443
    john@caret.cam.ac.uk 2005-12-09T13:32:29+00:00 re: lms/vle rants/comments
http://mbox.dr-chuck.net/sakai.devel/6/7 3586
    s-githens@northwestern.edu 2005-12-09T13:32:31-06:00 re: sakaiportallogin and presense
http://mbox.dr-chuck.net/sakai.devel/7/8 10600
    john@caret.cam.ac.uk 2005-12-09T13:42:24+00:00 re: lms/vle rants/comments
http://mbox.dr-chuck.net/sakai.devel/8/9 4892
    s-githens@northwestern.edu 2005-12-09T15:23:09-06:00 re: sakaiportallogin and presense
http://mbox.dr-chuck.net/sakai.devel/9/10 3206
    ys2n@virginia.edu 2005-12-09T17:54:17+00:00 sakaiportallogin and presense
http://mbox.dr-chuck.net/sakai.devel/10/11 4993
    ys2n@virginia.edu 2005-12-09T21:16:39+00:00 re: sakaiportallogin and presense
How many messages to retrieve?
```

The program scans content.sqlite from index 1 up to the first message number not already spidered and starts spidering at that message.
It continues spidering until it has spidered the desired number of messages or it reaches a page that does not appear to be a properly formatted message.
<br />
The content.sqlite data is pretty raw, with an inefficient data model, and not compressed.
It would be a bad idea to run any queries against this database as they would be slow.
<br />
The second process is running the program **model.py**.
model.py reads the rough/raw data from content.sqlite and produces a cleaned-up and well-modeled version of the data and stores it in the file **index.sqlite**.
The file index.sqlite will be much smaller than content.sqlite because it also compresses the header and body text.
Running model.py works as follows:

```
python model.py

Output:
Loaded allsenders: 1771 mapping: 29 dns mapping: 1
1 2005-12-08T23:34:30-06:00 ggolden22@mac.com
251 2005-12-22T10:03:20-08:00 tpamsler@ucdavis.edu
501 2006-01-12T11:17:34-05:00 lance@indiana.edu
751 2006-01-24T11:13:28-08:00 vrajgopalan@ucmerced.edu
1001 2006-02-02T08:27:30-07:00 john.ellis@rsmart.com
...
57751 2014-10-14T14:42:55+02:00 nguni52@gmail.com
58001 2014-11-10T20:45:51-05:00 neal.caidin@apereo.org
58251 2014-12-11T14:11:48+05:30 prabhu142003@gmail.com
58501 2015-01-21T11:18:36+01:00 mcarro@entornosdeformacion.com
58751 2015-02-21T19:44:31+11:00 steve.swinsburg@swinsborg.com
```

The model.py program does a number of data cleaning steps:
<br />
Domain names are truncated to two levels for .com, .org, .edu, and .net.
Other domain names are truncated to three levels.
So si.umich.edu becomes umich.edu and caret.cam.ac.uk becomes cam.ac.uk.
Also, mail addresses are forced to lower case and some of the @gmane.org address like "arwhyte-63aXycvo3TyHXe+LvDLADg@public.gmane.org" are converted to the real address whenever there is a matching real email address elsewhere in the message corpus.
<br />
If you look in the content.sqlite database, there are two tables that allow you to map both domain names and individual email addresses that change over the lifetime of the email list.
For example, Steve Githens used the following email addresses over the life of the Sakai developer list:
<br />
s-githens@northwestern.edu
sgithens@cam.ac.uk
swgithen@mtu.edu
<br />
We can add two entries to the Mapping table:
<br />
s-githens@northwestern.edu ->  swgithen@mtu.edu
sgithens@cam.ac.uk -> swgithen@mtu.edu
<br />
Thus, all the mail messages will be collected under one sender even if they used several email addresses over the lifetime of the mailing list.
You can also make similar entries in the DNSMapping table if there are multiple DNS names you want mapped to a single DNS.
In the Sakai data, the following mapping is added:
<br />
iupui.edu -> indiana.edu
<br />
So all the folks from the various Indiana University campuses are tracked together.
You can re-run the model.py over and over as you look at the data, and add mappings to make the data cleaner and cleaner.
When you are done, you will have a nicely indexed version of the email in index.sqlite.
This is the file to use to do data analysis.
With this file, data analysis will be really quick.
The first, simplest data analysis is to do a "Who does the most?" and "Which organization does the most?"
This is done using **basic.py**:

```
python basic.py

Output:
How many to dump? 10
Loaded messages= 58957 subjects= 29059 senders= 1765

Top 10 Email list participants:
steve.swinsburg@swinsborg.com 3301
azeckoski@unicon.net 1907
ian@cam.ac.uk 1591
csev@umich.edu 1468
david.horwitz@uct.ac.za 1221
matthew@longsight.com 1147
neal.caidin@apereo.org 931
stephen.marquard@uct.ac.za 926
adam.marshall@ox.ac.uk 888
arwhyte@umich.edu 868

Top 10 Email list organizations:
umich.edu 6782
gmail.com 5585
swinsborg.com 3301
cam.ac.uk 2626
uct.ac.za 2576
indiana.edu 2333
unicon.net 2305
longsight.com 2236
ox.ac.uk 1485
berkeley.edu 1388
```

There is a simple visualization of the word frequencies in the subject lines, in the file **word.py**:

```
python word.py

Output:
Range of counts: 43177 324
Output written to word.js
Open word.htm in a browser to see the vizualization
```

This produces the file **word.js** which you can visualize using the file **word.htm**.
<br />
A second visualization is in **line.py**.
It visualizes email participation by organizations over time.

```
python line.py

Output:
Loaded messages= 58957 senders= 1765
Top 10 Organizations:
['umich.edu', 'gmail.com', 'swinsborg.com', 'cam.ac.uk', 'uct.ac.za', 'indiana.edu', 'unicon.net', 'longsight.com', 'ox.ac.uk', 'berkeley.edu']
Output written to line.js
Open line.htm to visualize the data
```

This produces the file **line.js** which you can visualize using the file **line.htm**.

## Authors

* **Prathamesh Kumkar** - [iPrathamKumkar](https://github.com/iPrathamKumkar)

## Acknowledgments

* Special thanks to Mr. Charles Severance!
