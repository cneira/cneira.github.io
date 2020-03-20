```bash
make[1]: Leaving directory '/home/cneira/bpf-next/tools/lib/bpf'
 gcc -g -Wall -O2 -I../../../include/uapi -I../../../lib -I../../../lib/bpf -I../../../../include/generated -DHAVE_GENHDR -I../../../include -Dbpf_prog_load=bpf_prog_test_load -Dbpf_load_program=bpf_test_load_program -I. -I/home/cneira/bpf-next/tools/testing/selftests/bpf -Iverifier    test_verifier.c /home/cneira/bpf-next/tools/testing/selftests/bpf/test_stub.o /home/cneira/bpf-next/tools/testing/selftests/bpf/libbpf.a -lcap -lelf -lrt -lpthread -o /home/cneira/bpf-next/tools/testing/selftests/bpf/test_verifier
 test_verifier.c:25:10: fatal error: sys/capability.h: No such file or directory
    25 | #include 
       |          ^~~~~~
 compilation terminated.
 make: *** [../lib.mk:138: /home/cneira/bpf-next/tools/testing/selftests/bpf/test_verifier] Error 1
```  

This is solved by installing libcap-dev in Debian or libcap-devel in Fedora  
```bash  

 [cneira@ebpf-metal bpf]$ sudo dnf -y  search libcap-dev
 Last metadata expiration check: 0:05:49 ago on Fri 13 Sep 2019 06:09:10 PM -03.
 ====================================================================================== Name Matched: libcap-dev ======================================================================================
 libcap-devel.i686 : Development files for libcap
 libcap-devel.x86_64 : Development files for libcap
 [cneira@ebpf-metal bpf]$ sudo dnf -y  install libcap-devel
 Last metadata expiration check: 0:05:55 ago on Fri 13 Sep 2019 06:09:10 PM -03.
 Dependencies resolved.
  Package                                            Architecture                                 Version                                           Repository                                    Size
 Installing:
  libcap-devel                                       x86_64                                       2.26-5.fc30                                       fedora                                        26 k
 Transaction Summary
 Install  1 Package
 Total download size: 26 k
 Installed size: 16 k
 Downloading Packages:
 libcap-devel-2.26-5.fc30.x86_64.rpm    
 ```
 
