
## webdav-handler

A webdav handler for the Rust "hyper" HTTP server library. Uses a
interface similar to the Go x/net/webdav package:

- the library contains an HTTP handler (for Hyper 0.10.x at the moment)
- you supply a "filesystem" for backend storage, which can optionally
  implement reading/writing "DAV properties"
- you can supply a "locksystem" that handles the webdav locks

Currently passes the "basic", "copymove", "props", "locks" and "http"
checks of the Webdav Litmus Test testsuite. That's all of the base
RFC4918 webdav specification.

The litmus test suite also has tests for RFC3744 "acl" and "principal",
RFC5842 "bind", and RFC3253 "versioning". Those we do not support right now.

Included are two filesystems:
- localfs: serves a directory on the local filesystem
- memfs: ephemeral in-memory filesystem. supports DAV properties.

Also included are two locksystems:

- memls: ephemeral in-memory locksystem.
- fakels: fake locksystem. just enough LOCK/UNLOCK support for OSX/Windows.

# testing

```
cd src
RUST_LOG=webdav_handler=debug cargo run
```

This will start a server on port 4918, serving an in-memory filesystem.
For other options, run `cargo run -- --help`

# webdav protocol compliance

The standard for webdav compliance testing is "litmus", which is available
at https://github.com/tolsen/litmus .

You do not have to install the litmus binary, it's possible to run the tests
straight from the unpacked & compiled litmus directory:

```
$ TESTS="basic copymove props locks http" HTDOCS=htdocs TESTROOT=. ./litmus http://localhost:4918/

-> running `basic':
 0. init.................. pass
 1. begin................. pass
 2. options............... pass
 3. put_get............... pass
 4. put_get_utf8_segment.. pass
 5. mkcol_over_plain...... pass
 6. delete................ pass
 7. delete_null........... pass
 8. delete_fragment....... pass
 9. mkcol................. pass
10. mkcol_percent_encoded. pass
11. mkcol_again........... pass
12. delete_coll........... pass
13. mkcol_no_parent....... pass
14. mkcol_with_body....... pass
15. mkcol_forbidden....... pass
16. chk_ETag.............. pass
17. finish................ pass
<- summary for `basic': of 18 tests run: 18 passed, 0 failed. 100.0%
-> running `copymove':
 0. init.................. pass
 1. begin................. pass
 2. copy_init............. pass
 3. copy_simple........... pass
 4. copy_overwrite........ pass
 5. copy_nodestcoll....... pass
 6. copy_cleanup.......... pass
 7. copy_content_check.... pass
 8. copy_coll_depth....... pass
 9. copy_coll............. pass
10. depth_zero_copy....... pass
11. copy_med_on_coll...... pass
12. move.................. pass
13. move_coll............. pass
14. move_cleanup.......... pass
15. move_content_check.... pass
16. move_collection_check. pass
17. finish................ pass
<- summary for `copymove': of 18 tests run: 18 passed, 0 failed. 100.0%
-> running `props':
 0. init.................. pass
 1. begin................. pass
 2. propfind_invalid...... pass
 3. propfind_invalid2..... pass
 4. propfind_d0........... pass
 5. propinit.............. pass
 6. propfind_d1........... pass
 7. proppatch_invalid_semantics...................... pass
 8. propset............... pass
 9. propget............... pass
10. propfind_empty........ WARNING: Server did not return the property: displayname
                           WARNING: Server did not return the property: getcontentlanguage
    ...................... pass (with 2 warnings)
11. propfind_allprop_include...................... WARNING: Server did not return the property: displayname
                                                   WARNING: Server did not return the property: getcontentlanguage
                                                   WARNING: Server did not return the property: acl
                                                   WARNING: Server did not return the property: resource-id
    ...................... pass (with 4 warnings)
12. propfind_propname..... WARNING: Server did not return the property: displayname
                           WARNING: Server did not return the property: getcontentlanguage
                           WARNING: Server did not return the property: acl
                           WARNING: Server did not return the property: resource-id
    ...................... pass (with 4 warnings)
13. proppatch_liveunprotect...................... pass
14. propextended.......... pass
15. propcopy.............. pass
16. propget............... pass
17. propcopy_unmapped..... pass
18. propget............... pass
19. propmove.............. pass
20. propget............... pass
21. propdeletes........... pass
22. propget............... pass
23. propreplace........... pass
24. propget............... pass
25. propnullns............ pass
26. propget............... pass
27. prophighunicode....... pass
28. propget............... pass
29. propvalnspace......... pass
30. propwformed........... pass
31. propinit.............. pass
32. propmanyns............ pass
33. propget............... pass
34. property_mixed........ pass
35. propfind_mixed........ pass
36. propcleanup........... pass
37. finish................ pass
<- summary for `props': of 38 tests run: 38 passed, 0 failed. 100.0%
-> 10 warnings were issued.
-> running `http':
 0. init.................. pass
 1. begin................. pass
 2. expect100............. pass
 3. finish................ pass
<- summary for `http': of 4 tests run: 4 passed, 0 failed. 100.0%

```
