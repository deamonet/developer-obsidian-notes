This question shows research effort; it is useful and clear

I have read that `assume` will not run the test if assumption failed, but I am not sure regarding the logic of when to place `assert` vs `assume`.

For example: any resource loading check should be done with `assume`?

When should I use `assume` over `assert`?

(Note: i am looking for correct design of when to use one over the other)

-   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
-   [unit-testing](https://stackoverflow.com/questions/tagged/unit-testing "show questions tagged 'unit-testing'")
-   [junit](https://stackoverflow.com/questions/tagged/junit)

[Share](https://stackoverflow.com/q/44628483 "Short permalink to this question") Follow [edited Jun 19, 2017 at 11:31](https://stackoverflow.com/posts/44628483/revisions "show all edits to this post")

asked Jun 19, 2017 at 10:45

[Mike](https://stackoverflow.com/users/3083679/mike)


You would use `assume` if you have circumstances under which some tests should not run at all. "Not run" means that it cannot fail, because, well, it did not run.

You would use `assert` to fail a test if something goes wrong.

So, in a hypothetical scenario where:

-   you have different builds for different customers, and
-   you have some resource which is only applicable to a particular client, and
-   there is something testable about that resource, then

you would write a test which:

-   assumes that the resource is present, (so the test will not run on customers that do not have that resource,) and then
-   asserts that everything about the resource is okay (so on the customer that does actually have the resource, the test makes sure that the resource is as it should be.)