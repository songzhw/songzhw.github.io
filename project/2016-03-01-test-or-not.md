# Test? Or Not?

In the weekly supervisor meeting today, our front-end(html+js+css) supervisor argued with our QA supervisor with the test issues. They were doing a widely spread refactor (dozens of pages according to them). 

The front-end surpervisor （Let's call him "FES") wanted to submit their codes to QA, and let QA to do all the tests. His reason was "These pages are too many to test by us, the developer". 

And QA supervisor (Let's call him "QAS") did not agree with him. QAS said the developer should do some tests on these biggest issues and most important procedures (like paying procedures). 

Besides, QAS give us, the mobile team, praise. He said my team do a lot of test and passing all the smoking tests before submiting to the QA. As a result, our SDK project is very steady and has few bugs.

So, I want to talk about the quatity of our codes. The html codes is like our Android/iOS codes, which contains a lot of UI codes. I know, tests on UI is difficult. But, do we really can not do anything about it?

No. Here is some tips.

1. Test before submit

Only sumbit a little function every time. 

Test it (through auto test or your finger press) before you submit it to SVN/Git.

2. Write automatical test codes

If you are an Android developer, you can use **Espresso** to test your codes. Our project is using it, and the result is great. We have few bugs. 

Furthermore, we do not worried about the quatity of our codes when we do some refactor. The automatic test cases are our shield.

If you are an QA who tests Android apps/SDKs, you can use **UI Automator** to help you. UI Automator is a black box-style test framework. And the **UI Automator Viewer** is a good tool to help you.

3. Pass all the smoking tests

QA gives us some smoking tests, which covers the most importants issues about our new version. Developers **must** pass them all, then they are allowed to submit their product to the QA.

4. Pass all the tests before release

When the developers submitted products to the QA, and QA is testing, our developers are available to write all the Espreeso codes about the new features. 

Before the release of the product, developers should pass all the tests, old and new ones.

5. Jacoo

Jacoo is a good tool to help you figure out how many line of codes are covered by test cases. I ask my colleagues to cover 70% lines of the product code. 

Believe me, the "waste of time" is really worthy. It will help you to refactor, or to do some new requirements in the future. 


So, the process is easy:
1. developer write code
2. QA gives developers the smoking test cases
3. when developers complete their codes, they should pass all the smoking test cases.
4. Once passing all the smoking test cases, developer now can submit the product to QA
5. QA start testing. In the meantime, developers start to write Espresso tests.
6. Developers should pass all the Espresso tests before release.
7. Developers should make sure the coverage of test codes are above 70%.


