---
published: true
title: Yarssr is not loading the feed, how to fix it
layout: post
---
If you add a feed on yarssr and the icon doesn't turn green, it is time to enable the debug and look for the error.
First kill the running process:

`$ ps -fea | grep -i yarssr | grep -v grep | awk '{ print $2 }' | xargs kill -9`

Then, go to the app folder and start it back w/debug flag:

```shell
$ cd ~/.yarssr/;yarssr --debug
```

**HINT**, if you happen to have many feeds as me, it might be easier to clean up the config file to only leave the one that is not working. Do a back before that by reading my previous post [Yarssr deletes the feeds, how to deal with it][1].

[1]: http://emmanuel-galindo.github.io/2016/04/14/yarssr-deletes-the-feeds-how-to-deal-with-it.html

For me, it was printing:

```shell
[17:02:57] 
not well-formed (invalid token) at line 1, column 0, byte 0 at /usr/lib/x86_64-linux-gnu/perl5/5.20/XML/Parser.pm line 187.
```

This indicates the content of the URL is garbled, and it is not being handed correctly to the parser. 
In my computer, Yarssr is installed on /usr/share/yarssr and there you will find a module called Fetcher. 
By doing some inspections, it was returning the content without decoding it. 
The url that I am using (https://kat.cr/etc?rss=1) returns compressed content (use curl or wget)

```shell
$ curl -I https://kat.cr/
[...]
Content-Encoding: gzip
[...]
```

So, I've come up with my own _download method within Fetcher.pm. If you plan to use it:

1. Backup the current _download function to _download_old
+ Install Compress::Zlib if you haven't already. To check if you did: 

``` shell
$ perl -MCompress::Zlib
```

If you have to install it, just do

``` shell
$ cpan Compress::Zlib
```

**DISCLAIMER:** I haven't got the change to test against a protected RSS feed, therefore I have not tested the credentials part of the new procedure.

You will need the following moduled to get the procedure working.

```perl
use LWP::UserAgent;
use HTTP::Request;
use HTTP::Message;
use Compress::Zlib;
```

So, my own take of the _download procedure that will allow you to handle encoded content is:

```perl
sub _download {
        my ($url,$login) = @_;
        caller eq __PACKAGE__ or die;

        my $can_accept = HTTP::Message::decodable;
        my $agent = LWP::UserAgent->new(env_proxy => 1,keep_alive => 1, timeout => 30,'Accept-Encoding' => $can_accept,agent=>'Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.1');
        my $header = HTTP::Request->new(GET => $url);
        my $request = HTTP::Request->new('GET', $url, $header);
        my $response = $agent->request($request);

        if ($login->[0] and $login->[1]) {
                my $username = $login->[0];
                my $pass = $login->[1];
                $request->authorization_basic( "$username", "$pass" );
        }

        # Check the outcome of the response
        my ($content,$type);
        if ($response->is_success){
                ($type) = ($response->headers->{'content-type'}=~/(.*);.*/);
                #$ce = $response->headers->{'content-encoding'};
                $content = $response->decoded_content;
        }elsif ($response->is_error){
                return (0,$response->as_string);
        }
        return $content,$type;
}
```
