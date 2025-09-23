---
date: '2025-09-23T16:07:55-07:00'
draft: false
title: 'Week 14: ippeveprinter Configuration, DNS-SD, and HTTP'
---
I had more time this week to work on and debug the DNS-SD services and functionality of ippeveprinter. I was able to figure out how to get ippeveprinter recognized as a printer by CUPS, but I’ve also had a lot more issues with processing jobs.

First, I noticed that I have to load a `test.conf` file, or else seemingly every job will fail when running on my local machine. It turns out that the `load_ippserver_attributes` is very memory-intensive, and I was only able to get it to work after increasing a bunch of stack sizes in the kconfig (see [source code](https://github.com/HubertYGuan/zephyr-example/blob/main/prj.conf)). This means that I have only about 5% of my DRAM available for the heap, and I am also unsure if my pthread stacks are large enough to handle the HTTP functions either.

I think this is likely contributing towards the errors that I get when I try to test with libcups’ tools below:  
```
$ tools/ippfind-static -T 10 --exec tools/ipptool-static -V 2.0 -tIf examples/document-letter.pdf '{}' examples/ipp-2.0.test \;
ipptool: Unable to connect to "My\032Example\032Printer\032IPP.local" on port 8000: Connection timed out
"examples/ipp-1.1.test":
    RFC 8011 section 4.1.1: Bad request-id value 0                       [FAIL]
        RECEIVED: 111 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
    RFC 8011 section 4.1.4: No Operation Attributes                      [FAIL]
        RECEIVED: 119 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
    RFC 8011 section 4.1.4: attributes-charset                           [FAIL]
        RECEIVED: 122 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
    RFC 8011 section 4.1.4: attributes-natural-language                  [FAIL]
        RECEIVED: 122 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
    RFC 8011 section 4.1.4: attributes-natural-language + attributes-cha [FAIL]
        RECEIVED: 122 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
    RFC 8011 section 4.1.4: attributes-charset + attributes-natural-lang [FAIL]
        RECEIVED: 127 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
        EXPECTED: STATUS successful-ok (got server-error-internal-error)
        status-message="The printer or class does not exist."
        EXPECTED: printer-uri-supported
    RFC 8011 section 4.1.8: Unsupported IPP version 0.0                  [FAIL]
        RECEIVED: 125 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
    RFC 8011 section 4.2: No printer-uri operation attribute             [FAIL]
        RECEIVED: 119 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
    RFC 8011 section 4.2.1: Print-Job Operation                          [FAIL]
        RECEIVED: 127 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
        EXPECTED: STATUS successful-ok (got server-error-internal-error)
        EXPECTED: STATUS client-error-document-format-not-supported (got server-error-internal-error)
        EXPECTED: STATUS server-error-job-canceled (got server-error-internal-error)
        status-message="The printer or class does not exist."
        EXPECTED: job-uri
        EXPECTED: job-id
        EXPECTED: job-state
        EXPECTED: job-state-reasons
    RFC 8011 section 4.2.3: Validate-Job Operation                       [FAIL]
        RECEIVED: 127 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
        EXPECTED: STATUS successful-ok (got server-error-internal-error)
        status-message="The printer or class does not exist."
    RFC 8011 section 4.2.5: Get-Printer-Attributes Operation (default)   [FAIL]
        RECEIVED: 127 bytes in response
        status-code = server-error-internal-error (Invalid argument)
        Bad HTTP version (1.0)
        EXPECTED: STATUS successful-ok (got server-error-internal-error)
        status-message="The printer or class does not exist."
        EXPECTED: operations-supported
        EXPECTED: operations-supported
        EXPECTED: operations-supported
        EXPECTED: operations-supported
        EXPECTED: operations-supported
        EXPECTED: operations-supported
        EXPECTED: operations-supported
        EXPECTED: charset-configured
        EXPECTED: charset-supported
        EXPECTED: compression-supported
        EXPECTED: document-format-default
        EXPECTED: document-format-supported
        EXPECTED: generated-natural-language-supported
        EXPECTED: ipp-versions-supported
        EXPECTED: natural-language-configured
        EXPECTED: pdl-override-supported
        EXPECTED: printer-is-accepting-jobs
        EXPECTED: printer-name
        EXPECTED: printer-state
        EXPECTED: printer-state-reasons
        EXPECTED: printer-up-time
        EXPECTED: printer-uri-supported
        EXPECTED: queued-job-count
        EXPECTED: uri-authentication-supported
        EXPECTED: uri-security-supported
ipptool: Unable to connect to "My\032Example\032Printer\032IPP.local" on port 8000: Connection timed out
```
When I open my debugger and halt when I try to connect and print, I get these logs:
```
0x4208815d in http_create (host=0x0, port=0, addrlist=0x3fcdbdf8, family=0, encryption=HTTP_ENCRYPTION_IF_REQUESTED, blocking=true, mode=mode@entry=_HTTP_MODE_SERVER) at /home/(user)/zlibtest/modules/lib/libcups/cups/http.c:3530
3530      if (!host && mode == _HTTP_MODE_CLIENT)
(gdb) backtrace
#0  0x4208815d in http_create (host=0x0, port=0, addrlist=0x3fcdbdf8, family=0, encryption=HTTP_ENCRYPTION_IF_REQUESTED, blocking=true, 
    mode=mode@entry=_HTTP_MODE_SERVER) at /home/(user)/zlibtest/modules/lib/libcups/cups/http.c:3530
#1  0x420882fa in httpAcceptConnection (fd=7, blocking=true) at /home/(user)/zlibtest/modules/lib/libcups/cups/http.c:173
#2  0x4200ed89 in create_client (printer=0x3fce2458, sock=7) at /home/(user)/zlibtest/module-app/tests/libcups/ippeveprinter.h:850
#3  0x4200eed5 in run_printer (printer=0x3fce2458) at /home/(user)/zlibtest/module-app/tests/libcups/ippeveprinter.h:5705
#4  0x4200ff35 in ippeve_main (p1=<optimized out>) at /home/(user)/zlibtest/module-app/tests/libcups/ippeveprinter.h:496
#5  0x420260a1 in zephyr_thread_wrapper (arg1=0x0, arg2=0x4200fb88 <ippeve_main>, arg3=<optimized out>)
    at /home/(user)/zlibtest/zephyr/lib/posix/options/pthread.c:534
#6  0x42011e5c in z_thread_entry (entry=0x42026090 <zephyr_thread_wrapper>, p1=0x0, p2=0x4200fb88 <ippeve_main>, p3=0x0)
    at /home/(user)/zlibtest/zephyr/lib/os/thread_entry.c:48
(gdb) list
3525      _cups_globals_t *cg = _cupsGlobals(); // Thread global data
3526
3527
3528      DEBUG_printf("4http_create(host=\"%s\", port=%d, addrlist=%p, family=%d, encryption=%d, blocking=%s, mode=%d)", host, port, (void *)addrlist, family, encryption, blocking ? "true" : "false", mode);
3529
3530      if (!host && mode == _HTTP_MODE_CLIENT)
3531        return (NULL);
3532
3533      httpInitialize();
3534
```
And when continuing again and halting:
```
0x4005706c in ?? ()
(gdb) back
#0  0x4005706c in ?? ()
#1  0x42084ca6 in __memmove_ichk (len=<optimized out>, src=<optimized out>, dst=<optimized out>)
    at /home/(user)/zephyr-sdk-0.17.2/xtensa-espressif_esp32s3_zephyr-elf/xtensa-espressif_esp32s3_zephyr-elf/sys-include/ssp/string.h:84
#2  cupsArrayRemove (a=0x3fcdd640, e=<optimized out>) at /home/(user)/zlibtest/modules/lib/libcups/cups/array.c:721
#3  0x42092009 in _cupsStrFree (s=<optimized out>) at /home/(user)/zlibtest/modules/lib/libcups/cups/string.c:707
#4  0x42091dea in _cupsSetError (status=<optimized out>, message=0x3c10c670 "File exists", localize=false)
    at /home/(user)/zlibtest/modules/lib/libcups/cups/request.c:876
#5  0x420881c0 in http_create (host=<optimized out>, port=0, addrlist=<optimized out>, family=<optimized out>, encryption=HTTP_ENCRYPTION_IF_REQUESTED, 
    blocking=true, mode=mode@entry=_HTTP_MODE_SERVER) at /home/(user)/zlibtest/modules/lib/libcups/cups/http.c:3553
#6  0x420882fa in httpAcceptConnection (fd=7, blocking=true) at /home/(user)/zlibtest/modules/lib/libcups/cups/http.c:173
#7  0x4200ed89 in create_client (printer=0x3fce2458, sock=7) at /home/(user)/zlibtest/module-app/tests/libcups/ippeveprinter.h:850
#8  0x4200eed5 in run_printer (printer=0x3fce2458) at /home/(user)/zlibtest/module-app/tests/libcups/ippeveprinter.h:5705
--Type <RET> for more, q to quit, c to continue without paging--c
#9  0x4200ff35 in ippeve_main (p1=<optimized out>) at /home/(user)/zlibtest/module-app/tests/libcups/ippeveprinter.h:496
#10 0x420260a1 in zephyr_thread_wrapper (arg1=0x0, arg2=0x4200fb88 <ippeve_main>, arg3=<optimized out>) at /home/(user)/zlibtest/zephyr/lib/posix/options/pthread.c:534
#11 0x42011e5c in z_thread_entry (entry=0x42026090 <zephyr_thread_wrapper>, p1=0x0, p2=0x4200fb88 <ippeve_main>, p3=0x0) at /home/(user)/zlibtest/zephyr/lib/os/thread_entry.c:48
```
Furthermore, when I use CUPS 2.4.10 to print a test page, I get stuck on the following description for the job
```
processing since
<insert start time>
"Connected to printer."
```
And I also get the latter logs from GDB when I halt the program. (Also, on a sidenote, this is also not a fast process to debug since every time I do this, it seems I have to reset my access point to avoid DNS-SD resolution issues.)

# Next Steps

Since it looks like I’ve probably run out of memory when trying to make an HTTP connection with the client or in some HTTP function, I will try to save as much memory as possible from other parts of the program. However, for future memory concerns as well, I will also look into utilizing the ESP32 boards’ ability to load 4MB of external SPI RAM and then use that for dynamic allocations. After that, the other main hurdle will probably be the libcups HTTP API itself, if it needs some changes to work with Zephyr’s HTTP API. Since ippeveprinter doesn’t really process jobs (besides calling a command, but the command is not specified here), I believe that as long as ippeveprinter can receive the test page data and communicate over HTTP with the printing client, it should be able to “print” test pages and hopefully pass the tests provided. As always, you can check on my progress in the project proposal and design diagram below.