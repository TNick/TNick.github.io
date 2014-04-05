---
layout: post
title:  "proftpd uploads"
date:   2014-04-05 13:00:00
categories: walkthrough
---

Overview
--------

The [proftpd](http://www.proftpd.org/) is a 
highly configurable GPL-licensed FTP server.
In Ubuntu, it can be installed using:

{% highlight bash %}
sudo apt-get install proftpd
{% endhighlight %}

The configuration may be changed with:

{% highlight bash %}
sudo nano /etc/proftpd/proftpd.conf
# syntax check the file
proftpd -t -d5
{% endhighlight %}

According to 
[help page](http://www.proftpd.org/docs/howto/Stopping.html) 
on their site, proftpd team recommends stopping the
server using:

{% highlight bash %}
# location may be different on your system
# like /run/proftpd.pid
kill -TERM `cat /usr/local/var/proftpd.pid`
# don't use 
# sudo service proftpd restart
{% endhighlight %}

The status may be checked using 

{% highlight bash %}
/etc/init.d/proftpd status
# returns something like
# ProFTPD is started in standalone mode, currently running.
{% endhighlight %}

To start and stop the server via init.d script use:

{% highlight bash %}
sudo /etc/init.d/proftpd start
sudo /etc/init.d/proftpd stop
{% endhighlight %}


Acting on upload
----------------

ProFTPD has a module - 
[mod_exec](http://www.castaglia.org/proftpd/modules/mod_exec.html) -
that is not compiled by default. The authors do not recommend
using it for reacting on uploads. Instead, the FIFO way should be
used, as explained in 
[Logging](http://www.proftpd.org/docs/howto/Logging.html).

Another approach is the one involving named pipes as used in
[ftpmail](http://www.proftpd.org/docs/contrib/ftpmail.html). This is
a perl script that can be found in `contrib` directory inside
proftpd source package.

This (ftpmail) is the script that I modified to invoke a program with
following arguments (see the bottom of this post for full script):

{% highlight bash %}
$program \
  "$full_path_to_file" \
  "$date_and_time_of_transfer" \
  "$durration_of_transfer" \
  "$size_in_bytes" \
  "$transfer_status" \
  "$transfer_type"
{% endhighlight %}

`transfer_status` may be either "Completed" or "Incomplete".
`transfer_type` may be either "Binary" or "ASCII".


We need to change `/etc/init.d/proftpd`, with the `start()` routine
creating the named pipe and starting `onftpupload` script
before starting `proftpd`:

{% highlight bash %}
# ...
# at the beginning of the file, after PIDFILE
# is set (line 5)

# we save the PID of our script here;
# this may be something like /run/proftpd.pid.onu
ONUPIDFILE=$PIDFILE.onu

# named pipe can't reside in world writeable directory
ONUFIFO=/var/log/proftpd/onftpupload.fifo

# custom script derived from ftpmail
ONUSCRIPT=/usr/sbin/onftpupload

# the program that receives input
ONUAPP=/usr/sbin/onftpuploadapp

# ...

start()
{
    log_daemon_msg "Starting ftp server" "$NAME"
    
    # create a named pipe
    mkfifo "$ONUFIFO"
    
    # start listening to named pipe
    "$ONUSCRIPT"    \
         --fifo=$ONUFIFO \
         --app=$ONUAPP \
         --log=/var/log/proftpd/xferlog \
         --sleep=0.5 &

    # save the pid of the script so we can kill it
    echo $! > "$ONUPIDFILE"

    start-stop-daemon ...
    
	# ...

}

stop_onu()
{
    kill -9 `cat $ONUPIDFILE`
    rm -f "$ONUPIDFILE"
    rm -f "$ONUFIFO"
}

signal()
{
    # ...
    if [ "$SIGNAL" = "KILL" ]; then
        rm -f "$PIDFILE"
        stop_onu
    fi
    if [ "$SIGNAL" = "TERM" ]; then
        stop_onu
    fi
    
	#...
    
{% endhighlight %}

`/etc/proftpd/proftpd.conf` (and not `/etc/proftpd.conf`)
also needs to be changed to
let proftpd know about our fifo file:

{% highlight bash %}
TransferLog  /var/log/proftpd/onftpupload.fifo
{% endhighlight %}

To test the functionality a simple bash file
can be created (`/usr/sbin/onftpuploadapp`):

{% highlight bash %}
#/bin/bash
echo "Tester program for proftpd named pipe" >> /tmp/proftptest
for i in $*; do
  echo "$i"  >> /tmp/onftpuploadtest
done
{% endhighlight %}

----

Here is `/usr/sbin/onftpupload`:

{% highlight perl %}

#!/usr/bin/env perl
# ---------------------------------------------------------------------------
# Copyright (C) 2008-2013 TJ Saunders <tj@castaglia.org>
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Suite 500, Boston, MA 02110-1335, USA.
#
#  $Id: ftpmail,v 1.8 2013/10/13 22:34:14 castaglia Exp $
# ---------------------------------------------------------------------------
# Original script was modified by Nicu Tofan <nicu.tofan@gmail.com>
# to send information about uploads to a local application. Same 
# license conditions apply.
# ---------------------------------------------------------------------------

use strict;
use warnings;

use File::Basename qw(basename);
use Getopt::Long;
use MIME::Base64 qw(encode_base64);
use Time::HiRes qw(usleep);

my $program = basename($0);

my $opts = {};
GetOptions($opts, 
  'fifo=s', 
  'app=s', 
  'help', 
  'ignore-users=s',
  'log=s', 
  'sleep=s');

if ($opts->{help}) {
  usage();
  exit 0;
}

unless ($opts->{fifo}) {
  print STDERR "$program: missing required --fifo parameter\n";
  exit 1;
}
my $fifo = $opts->{fifo};

unless ($opts->{app}) {
  print STDERR "$program: missing required --app parameter\n";
  exit 1;
}
my $app = $opts->{app};

my $delay = 0.5;
if ($opts->{sleep}) {
  $delay = $opts->{sleep};
}

my $fifoh;
if (open($fifoh, "< $fifo")) {

  while (1) {
    my $line = <$fifoh>;
    if ($line) {
      chomp($line);

      if ($line =~ /^(\S+\s+\S+\s+\d+\s+\d+:\d+:\d+\s+\d+)\s+(\d+)\s+(.*?)\s+(\d+)\s+(.*?)\s+(\S+)\s+(\S+)\s+(\S+)\s+(\S+)\s+(.*?)\s+.*?(\S+)$/o) {
        my $curr_time = $1;
        my $xfer_nsecs = $2;
        my $client = $3;
        my $nbytes = $4;

        # Note that any spaces or control characters will be replaced in this
        # path with underscores.  This can make finding the actual file, as for
        # attachments, rather difficult; we have to test to find the difference
        # between a real underscore in the name, and a substituted underscore.
        my $path = $5;

        unless (-e $path) {
          # Perform a quick-and-dirty check, on the assumption that all of the
          # underscores in the given path are actually spaces.  If a
          # combination of underscores and spaces appears in the real file,
          # we won't detect that here.

          my $alt_path = $path;
          $alt_path =~ s/_/ /g;

          if (-e $alt_path) {
            $path = $alt_path;
          }
        }

        my $xfer_type = $6;
        my $action_flag = $7;
        my $xfer_direction = $8;
        my $access_mode = $9;
        my $user_name = $10;
        my $completion_status = $11;

        my $send_email = 0;
        if ($xfer_direction eq 'i') {
          $send_email = 1;

          if ($opts->{'ignore-users'}) {
            if ($user_name =~ /$opts->{'ignore-users'}/) {
              $send_email = 0;
            }
          }
        }

        if ($send_email) {
          send_email({
            timestamp => $curr_time,
            duration => $xfer_nsecs,
            client => $client,
            size => $nbytes,
            file => $path,
            transfer_type => $xfer_type,
            auth_mode => $access_mode,
            user => $user_name,
            status => $completion_status,
            direction => $xfer_direction,
          });
        }
      }

      if ($opts->{log}) {
        # Note: since this opens, writes, then closes the log file for every
        # write, it will interact with log rotation scripts MUCH better than
        # proftpd by itself.  Just one of the small benefits.

        my $log_file = $opts->{log};
        my $logfh;

        if (open($logfh, ">> $log_file")) {
          print $logfh "$line\n";

          unless (close($logfh)) {
            print STDERR "$program: error writing to log file '$log_file': $!\n";
          }

        } else {
          print STDERR "$program: error opening log file '$log_file': $!\n";
        }
      }

    } else {
      # No input at this time.  Sleep for half a second (or less) and check
      # again.
      usleep($delay * 1000000);
    }
  }

  close($fifoh);

} else {
  die "$program: unable to read FIFO '$fifo': $!\n";
}

sub send_email {
  my $transfer_info = shift;

  my $file = $transfer_info->{file};
  my $file_str = basename($file);

  my $transferred = 'uploaded';
  if ($transfer_info->{direction} eq 'o') {
    $transferred = 'downloaded';
  }

  my $bytes = $transfer_info->{size};
  my $timestamp = $transfer_info->{timestamp};

  my $status = "Completed";
  if ($transfer_info->{status} eq 'i') {
    $status = "Incomplete";
  }

  my $secs = $transfer_info->{duration};

  my $type_str = "Binary";
  if ($transfer_info->{transfer_type} eq 'a') {
    $type_str = "ASCII";
  }

  my $app_status = system("$app", 
    "\"$file\"", 
    "\"$timestamp\"", 
	"\"$secs\"", 
	"\"$bytes\"", 
	"\"$status\"",  
	"\"$type_str\""
	);
}

sub usage {
  print <<EOH;

usage: $program [--help] [--fifo \$path] [--app \$path] [--log \$path]
  [--ignore-users \$regex] [--sleep \$time]

The purpose of this script is to monitor the TransferLog written by proftpd
for uploaded files.  Whenever a file is uploaded by a user, an program
is called.  The program receives the timestamp,
the name of the user who uploaded the file, the path to the uploaded file, the
size of the uploaded file, and the time it took to upload.

Command-line options:

  --app \$path		Indicates the path to the application that
			will be invoked for each uploaded file.
			This parameter is REQUIRED.

  --fifo \$path		Indicates the path to the FIFO to which proftpd is
			writing its TransferLog.  That is, this is the path
			that you used for the TransferLog directive in your
			proftpd.conf.  This parameter is REQUIRED.
			
  --help		Displays this message.

  --ignore-users \$regex
			Specifies a Perl regular expression.  If the uploading
			user name matches this regular expression, then NO
			notification is sent.

  --log \$path		Since this script reads the TransferLog using FIFOs,
			the actual TransferLog file is not written by default.
			Use this option to write the normal TransferLog file,
			in addition to watching for uploads.

  --sleep \$time	to sleep between reads; by default this is half a 
			second (0.5).

EOH
}

{% endhighlight %}

The script will continue to log traffic at
`/var/log/proftpd/xferlog`. Rotation of log file
can (and should) be done manually, as the logging
does not go trough syslog.
