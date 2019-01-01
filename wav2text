#!/usr/bin/perl

# (c) 2018 by Matthias Walliczek, matthias@walliczek.de
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

use IPC::Open2;
use strict;
use warnings;
use JSON qw( decode_json );
use utf8;

my $keyfile = $ENV{'HOME'} . "/.google_speech_api_key";
my $key;

open KEYFILE, "<", $keyfile or die "Konnte $keyfile nicht oeffnen. $!\n";
if (!($key = <KEYFILE>)) {
  die "$keyfile enthaehlt keinen Text!";
}

my $content = `sox -t wav - -b 16 -e signed-integer --endian little -t wav - | base64 -w 0`;

my $postcontent = qq{
{
  "config": {
    "encoding": "LINEAR16",
    "languageCode": "de-DE"
  },
  "audio": {
    "content": "$content"
  }
}
};

my ($rh, $wh);
my $pid = open2($rh, $wh, 'curl -sS -X POST -H "Content-Type: application/json; charset=utf-8" --data-binary @- https://speech.googleapis.com/v1/speech:recognize?key=' . $key . ' 2>/dev/null') or die "open2() failed $!";
print $wh $postcontent;
close($wh);

my $resultJson = do { local $/; <$rh> };

waitpid $pid, 0;

my $result = decode_json( $resultJson)->{'results'}[0]{'alternatives'}[0]{'transcript'};

utf8::encode($result);

print $result;
