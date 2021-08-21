\#!/usr/bin/perl
###############################################################################

\# Program  : etlslave_unix.pl
\# Argument : (ControlFile)
\#            This program need one argument.
###############################################################################

use strict;
use Mojo::UserAgent;
use Mojo::URL;
use etl_unix;
use IPC::SysV qw(S_IRWXU IPC_CREAT);
use IPC::Semaphore ();
use POSIX qw(:sys_wait_h);

my $AUTO_HOME = $ENV{AUTO_HOME};

if (!defined($AUTO_HOME) || $AUTO_HOME eq '') {
   print "AUTO_HOME not defined, program terminated!\n";
   exit 1;
}

$ENV{'PGCLIENTENCODING'} = 'UTF8';

my $LOGDIR;
my $LOGFILE;
my $TODAY;

my $TRUE  = $ETL::TRUE;
my $FALSE = $ETL::FALSE;
my $max_count = $ETL::JOBPROCESS;

my $STOP_FLAG = 0;
my $LOCK_FILE;

my $ControlFile;
my $Server = $ETL::ETL_SERVER;
my ($Sys, $Job, $AutoOff, $JobType, $TxDate, $TxHour, $TxMin, $ProjectId, $TimeTrigger, $CheckCalendar);
my ($Action, $RetryDelay, $MaxRetryTimes, $RetryTimes, $RetryAtFailedStep, $Frequency);
my ($ipaddr, $port, $httpport, $primary);
my $JobSessionID;
my $ProcessID;
my $txdate;
my $MaxDuration;

my $SyntaxMode = 0;
my $TriggerEvent = 1;

my @dirFileList;
my $dbCon;

my @dataFileList;
my %env;
my ($ua, $log1, $log2, $rs);

my $semkey = $$;
my ($sem, $pid);
my %childs;