# Overengineering Testing


## **Sign One: You Automate EVERYTHING**

 \n Not everything should be automated. Some tests are very difficult to automate, and those tests are the ones that are the most likely to be flaky and require constant maintenance. You should always be doing some manual exploratory testing anyway, so why not just keep that tricky test as a manual test and save yourself time and aggravation?


## **Sign Two: Asserting On Too Many Things in Your Tests**

 \n As the great Nikolay Advolodkin states, “For UI automation, every single step is a chance for something to go wrong.” This is why he recommends atomic tests: tests that check only one thing, and that run very quickly. It’s better to have a hundred small tests that each take less than thirty seconds to run than to have ten tests that each take five minutes to run. With the small tests, you will be much less likely to have false failures.


## **Sign Three: Abstracting Out Too Much of Your Test Code**

 \n It’s a great idea to observe the “Don’t Repeat Yourself” rule of clean code, but it can be taken to the extreme. If you have a test that ends with something like this: \n `assertErrorMessage()` \n consider that whoever maintains this test is going to have to search for this method to find out what the test actually does. Something like this could be much more maintainable: \n `cy.get('#error').invoke('text').should('contain', 'card is expired')` \n This tells the reader exactly what the expected error should be and where it should be found.



## **Sign Four: Constantly Rewriting Your Tests with the Latest Automation Tool**

 \n It’s great to update your test code periodically, but if your tests are running and passing, there’s no real reason to switch to a new testing tool. It just makes more work for you and for everyone who needs to maintain your tests. There’s an American expression: “If it ain’t broke, don’t fix it!” This means that if something is working, there’s no need to change it or replace it with something else. If you find that your tests have become flaky or difficult to maintain, then by all means look for a new tool. Otherwise, be thankful for the tool you have.

The primary goal of test automation is that it should provide a way for you to do automated checks on your product’s code reliably and efficiently. Your code should also be readable and maintainable. Anything that doesn’t meet those goals is likely over-engineering!