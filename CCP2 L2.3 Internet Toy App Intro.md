GitHub Repo Search Code
The code for this app can be found in the Lesson02-GitHub-Repo-Search folder of the Toy App Repository.

If you need a refresher on how the code is organized, please refer the concept where we introduced the code flow.

Explanation of GitHub Repo Search
The Github Repo Search app searches for a GitHub repository by name. The URL you'll use to get search information will be something like the URL below, which searches for repositories containing the word android and sorts by the number of stars the repo has:

https://api.github.com/search/repositories?q=android&sort=stars

Which returns information in JSON. We'll be going over how to make sense of this returned JSON, parse it and display it in your app during this lesson. We'll also cover how to connect to the internet and download data. Let's get started!

