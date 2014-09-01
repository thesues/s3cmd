## S3cmd tool for Amazon Simple Storage Service (S3)


* Author: Michal Ludvig, michal@logix.cz
* [Project homepage](http://s3tools.org)
* (c) [TGRMN Software](http://www.tgrmn.com) and contributors


S3tools / S3cmd mailing lists:

* Announcements of new releases: s3tools-announce@lists.sourceforge.net
* General questions and discussion: s3tools-general@lists.sourceforge.net
* Bug reports: s3tools-bugs@lists.sourceforge.net

### What is S3cmd

S3cmd (`s3cmd`) is a free command line tool and client for uploading, retrieving and managing data in Amazon S3 and other cloud storage service providers that use the S3 protocol, such as Google Cloud Storage or DreamHost DreamObjects. It is best suited for power users who are familiar with command line programs. It is also ideal for batch scripts and automated backup to S3, triggered from cron, etc.

S3cmd is written in Python. It's an open source project available under GNU Public License v2 (GPLv2) and is free for both commercial and private use. You will only have to pay Amazon for using their storage.

Lots of features and options have been added to S3cmd, since its very first release in 2008.... we recently counted more than 60 command line options, including multipart uploads, encryption, incremental backup, s3 sync, ACL and Metadata management, S3 bucket size, bucket policies, and more!

### What is Amazon S3

Amazon S3 provides a managed internet-accessible storage service where anyone can store any amount of data and retrieve it later again.

S3 is a paid service operated by Amazon. Before storing anything into S3 you must sign up for an "AWS" account (where AWS = Amazon Web Services) to obtain a pair of identifiers: Access Key and Secret Key. You will need to
give these keys to S3cmd. Think of them as if they were a username and password for your S3 account.

### Amazon S3 pricing explained

At the time of this writing the costs of using S3 are (in USD):

$0.15 per GB per month of storage space used

plus

$0.10 per GB - all data uploaded

plus

$0.18 per GB - first 10 TB / month data downloaded
$0.16 per GB - next 40 TB / month data downloaded
$0.13 per GB - data downloaded / month over 50 TB

plus

$0.01 per 1,000 PUT or LIST requests
$0.01 per 10,000 GET and all other requests

If for instance on 1st of January you upload 2GB of photos in JPEG from your holiday in New Zealand, at the end of January you will be charged $0.30 for using 2GB of storage space for a month, $0.20 for uploading 2GB of data, and a few cents for requests. That comes to slightly over $0.50 for a complete backup of your precious holiday pictures.

In February you don't touch it. Your data are still on S3 servers so you pay $0.30 for those two gigabytes, but not a single cent will be charged for any transfer. That comes to $0.30 as an ongoing cost of your backup. Not too bad.

In March you allow anonymous read access to some of your pictures and your friends download, say, 500MB of them. As the files are owned by you, you are responsible for the costs incurred. That means at the end of March you'll be charged $0.30 for storage plus $0.09 for the download traffic generated by your friends.

There is no minimum monthly contract or a setup fee. What you use is what you pay for. At the beginning my bill used to be like US$0.03 or even nil.

That's the pricing model of Amazon S3 in a nutshell. Check the [Amazon S3 homepage](http://aws.amazon.com/s3) for more details.

Needless to say that all these money are charged by Amazon itself, there is obviously no payment for using S3cmd :-)

### Amazon S3 basics

Files stored in S3 are called "objects" and their names are officially called "keys". Since this is sometimes confusing for the users we often refer to the objects as "files" or "remote files". Each object belongs to exactly one "bucket".

To describe objects in S3 storage we invented a URI-like schema in the following form:

```
s3://BUCKET
```
or

```
s3://BUCKET/OBJECT
```

### Buckets

Buckets are sort of like directories or folders with some restrictions:

1. each user can only have 100 buckets at the most,
2. bucket names must be unique amongst all users of S3,
3. buckets can not be nested into a deeper hierarchy and
4. a name of a bucket can only consist of basic alphanumeric
   characters plus dot (.) and dash (-). No spaces, no accented
   or UTF-8 letters, etc.

It is a good idea to use DNS-compatible bucket names. That for instance means you should not use upper case characters. While DNS compliance is not strictly required some features described below are not available for DNS-incompatible named buckets. One more step further is using a fully qualified domain name (FQDN) for a bucket - that has even more benefits.

* For example "s3://--My-Bucket--" is not DNS compatible.
* On the other hand "s3://my-bucket" is DNS compatible but
  is not FQDN.
* Finally "s3://my-bucket.s3tools.org" is DNS compatible
  and FQDN provided you own the s3tools.org domain and can
  create the domain record for "my-bucket.s3tools.org".

Look for "Virtual Hosts" later in this text for more details regarding FQDN named buckets.

### Objects (files stored in Amazon S3)

Unlike for buckets there are almost no restrictions on object names. These can be any UTF-8 strings of up to 1024 bytes long. Interestingly enough the object name can contain forward slash character (/) thus a `my/funny/picture.jpg` is a valid object name. Note that there are not directories nor buckets called `my` and `funny` - it is really a single object name called `my/funny/picture.jpg` and S3 does not care at all that it _looks_ like a directory structure.

The full URI of such an image could be, for example:

```
s3://my-bucket/my/funny/picture.jpg
```

### Public vs Private files

The files stored in S3 can be either Private or Public. The Private ones are readable only by the user who uploaded them while the Public ones can be read by anyone. Additionally the Public files can be accessed using HTTP protocol, not only using `s3cmd` or a similar tool.

The ACL (Access Control List) of a file can be set at the time of upload using `--acl-public` or `--acl-private` options with `s3cmd put` or `s3cmd sync` commands (see below).

Alternatively the ACL can be altered for existing remote files with `s3cmd setacl --acl-public` (or `--acl-private`) command.

### Simple s3cmd HowTo

1) Register for Amazon AWS / S3

Go to http://aws.amazon.com/s3, click the "Sign up for web service" button in the right column and work through the registration. You will have to supply your Credit Card details in order to allow Amazon charge you for S3 usage. At the end you should have your Access and Secret Keys.

2) Run `s3cmd --configure`

You will be asked for the two keys - copy and paste them from your confirmation email or from your Amazon account page. Be careful when copying them! They are case sensitive and must be entered accurately or you'll keep getting errors about invalid signatures or similar.

Remember to add ListAllMyBuckets permissions to the keys or you will get an AccessDenied error while testing access.

3) Run `s3cmd ls` to list all your buckets.

As you just started using S3 there are no buckets owned by you as of now. So the output will be empty.

4) Make a bucket with `s3cmd mb s3://my-new-bucket-name`

As mentioned above the bucket names must be unique amongst _all_ users of S3. That means the simple names like "test" or "asdf" are already taken and you must make up something more original. To demonstrate as many features as possible let's create a FQDN-named bucket `s3://public.s3tools.org`:

```
$ s3cmd mb s3://public.s3tools.org

Bucket 's3://public.s3tools.org' created
```

5) List your buckets again with `s3cmd ls`

Now you should see your freshly created bucket:

```
$ s3cmd ls

2009-01-28 12:34  s3://public.s3tools.org
```

6) List the contents of the bucket:

```
$ s3cmd ls s3://public.s3tools.org
$
```

It's empty, indeed.

7) Upload a single file into the bucket:

```
$ s3cmd put some-file.xml s3://public.s3tools.org/somefile.xml

some-file.xml -> s3://public.s3tools.org/somefile.xml  [1 of 1]
 123456 of 123456   100% in    2s    51.75 kB/s  done
```

Upload a two-directory tree into the bucket's virtual 'directory':

```
$ s3cmd put --recursive dir1 dir2 s3://public.s3tools.org/somewhere/

File 'dir1/file1-1.txt' stored as 's3://public.s3tools.org/somewhere/dir1/file1-1.txt' [1 of 5]
File 'dir1/file1-2.txt' stored as 's3://public.s3tools.org/somewhere/dir1/file1-2.txt' [2 of 5]
File 'dir1/file1-3.log' stored as 's3://public.s3tools.org/somewhere/dir1/file1-3.log' [3 of 5]
File 'dir2/file2-1.bin' stored as 's3://public.s3tools.org/somewhere/dir2/file2-1.bin' [4 of 5]
File 'dir2/file2-2.txt' stored as 's3://public.s3tools.org/somewhere/dir2/file2-2.txt' [5 of 5]
```

As you can see we didn't have to create the `/somewhere` 'directory'. In fact it's only a filename prefix, not a real directory and it doesn't have to be created in any way beforehand.

8) Now list the bucket's contents again:

```
$ s3cmd ls s3://public.s3tools.org

                       DIR   s3://public.s3tools.org/somewhere/
2009-02-10 05:10    123456   s3://public.s3tools.org/somefile.xml
```

Use --recursive (or -r) to list all the remote files:

```
$ s3cmd ls --recursive s3://public.s3tools.org

2009-02-10 05:10    123456   s3://public.s3tools.org/somefile.xml
2009-02-10 05:13        18   s3://public.s3tools.org/somewhere/dir1/file1-1.txt
2009-02-10 05:13         8   s3://public.s3tools.org/somewhere/dir1/file1-2.txt
2009-02-10 05:13        16   s3://public.s3tools.org/somewhere/dir1/file1-3.log
2009-02-10 05:13        11   s3://public.s3tools.org/somewhere/dir2/file2-1.bin
2009-02-10 05:13         8   s3://public.s3tools.org/somewhere/dir2/file2-2.txt
```

9) Retrieve one of the files back and verify that it hasn't been
   corrupted:

```
$ s3cmd get s3://public.s3tools.org/somefile.xml some-file-2.xml

s3://public.s3tools.org/somefile.xml -> some-file-2.xml  [1 of 1]
 123456 of 123456   100% in    3s    35.75 kB/s  done
```

```
$ md5sum some-file.xml some-file-2.xml

39bcb6992e461b269b95b3bda303addf  some-file.xml
39bcb6992e461b269b95b3bda303addf  some-file-2.xml
```

Checksums of the original file matches the one of the retrieved ones. Looks like it worked :-)

To retrieve a whole 'directory tree' from S3 use recursive get:

```
$ s3cmd get --recursive s3://public.s3tools.org/somewhere

File s3://public.s3tools.org/somewhere/dir1/file1-1.txt saved as './somewhere/dir1/file1-1.txt'
File s3://public.s3tools.org/somewhere/dir1/file1-2.txt saved as './somewhere/dir1/file1-2.txt'
File s3://public.s3tools.org/somewhere/dir1/file1-3.log saved as './somewhere/dir1/file1-3.log'
File s3://public.s3tools.org/somewhere/dir2/file2-1.bin saved as './somewhere/dir2/file2-1.bin'
File s3://public.s3tools.org/somewhere/dir2/file2-2.txt saved as './somewhere/dir2/file2-2.txt'
```

Since the destination directory wasn't specified, `s3cmd` saved the directory structure in a current working directory ('.').

There is an important difference between:

```
get s3://public.s3tools.org/somewhere
```

and

```
get s3://public.s3tools.org/somewhere/
```

(note the trailing slash)

`s3cmd` always uses the last path part, ie the word after the last slash, for naming files.

In the case of `s3://.../somewhere` the last path part is 'somewhere' and therefore the recursive get names the local files as somewhere/dir1, somewhere/dir2, etc.

On the other hand in `s3://.../somewhere/` the last path
part is empty and s3cmd will only create 'dir1' and 'dir2'
without the 'somewhere/' prefix:

```
$ s3cmd get --recursive s3://public.s3tools.org/somewhere /tmp

File s3://public.s3tools.org/somewhere/dir1/file1-1.txt saved as '/tmp/dir1/file1-1.txt'
File s3://public.s3tools.org/somewhere/dir1/file1-2.txt saved as '/tmp/dir1/file1-2.txt'
File s3://public.s3tools.org/somewhere/dir1/file1-3.log saved as '/tmp/dir1/file1-3.log'
File s3://public.s3tools.org/somewhere/dir2/file2-1.bin saved as '/tmp/dir2/file2-1.bin'
```

See? It's `/tmp/dir1` and not `/tmp/somewhere/dir1` as it was in the previous example.

10) Clean up - delete the remote files and remove the bucket:

Remove everything under s3://public.s3tools.org/somewhere/

```
$ s3cmd del --recursive s3://public.s3tools.org/somewhere/

File s3://public.s3tools.org/somewhere/dir1/file1-1.txt deleted
File s3://public.s3tools.org/somewhere/dir1/file1-2.txt deleted
...
```

Now try to remove the bucket:

```
$ s3cmd rb s3://public.s3tools.org

ERROR: S3 error: 409 (BucketNotEmpty): The bucket you tried to delete is not empty
```

Ouch, we forgot about `s3://public.s3tools.org/somefile.xml`. We can force the bucket removal anyway:

```
$ s3cmd rb --force s3://public.s3tools.org/

WARNING: Bucket is not empty. Removing all the objects from it first. This may take some time...
File s3://public.s3tools.org/somefile.xml deleted
Bucket 's3://public.s3tools.org/' removed
```

### Hints

The basic usage is as simple as described in the previous section.

You can increase the level of verbosity with `-v` option and if you're really keen to know what the program does under its bonnet run it with `-d` to see all 'debugging' output.

After configuring it with `--configure` all available options are spitted into your `~/.s3cfg` file. It's a text file ready to be modified in your favourite text editor.

For more information refer to the [S3cmd / S3tools homepage](http://s3tools.org).

### License

Copyright (C) 2014 TGRMN Software - http://www.tgrmn.com - and contributors

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.
