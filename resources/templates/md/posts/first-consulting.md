{
 :title "Lessons learned from the software consulting job"
 :layout :post
 :date "2019-06-23"
 :tags ["Clojure", "Consulting"]
}

I live in Taiwan and I can not find Clojure jobs here. Although the first legal gay wedding in Asia took place here, it seems that the real programming language innovation still needs some evangelists to spread it. Therefore, I decided to create Clojure job by myself in January this year: I had a chance to develop enterprise software for the LINE Taiwan, and I chose Clojure as my primary technical stack.

## I should discuss about technical stack even before we signed the contract.

When I discussed with my client about this enterprise software solution, we focused on the problem domain. However, when I told my clients that I want to use Clojure, Datomic, and ClojureScript, my clients wanted to say no. They said a lot of cliches like they never hear Clojure before, difficult to find Clojure programmers. Finally, I made some compromises: I still used React with javascript in frontend, but I used Datomic and Clojure on backend. I proposed the reason that their business requirements have temporal queries which will be like a piece of cake for Datomic but very time-consuming for traditional relational databases.

After developing this project for a while, I regretted that I did not insist on ClojureScript. I really spent some time on javascript boilerplate code, and the time spent did not bring any value to my clients.

## A very simple user login is good enough for a small group of users

The enterprise software solution needs to be an on-premise solution, installed on the private network at LINE offices. There will be about 30 users login everyday to use this software. At the beginning, I thought three different ways to solve the user login problems:

1. Single signed-on with other enterprise software in LINE
2. Leverage third party Authorization service
3. Traditional user login backend APIs and frontend UI with login/register/user management functions like resetting password.

Option 2 might be fast enough, but my clients did not like third party service.

My final proposal was a login module like this:
1. Frontend UI provided the login and password modification functions to ordinary users.
2. The administrator of this system used ETL (extract-transform-load) to manage user accounts. Given this design, we did not need any user registration or user accounts management UI.

## Revenue spreading problem seems to be koan in 4clojure.com

There was a business requirement, I called it as revenue spreading problem, in this enterprise software.

Revenue spreading problem:
1. For every order, there is a start date and end date of this order. The total days of an order are `(end date - start date + 1)`
2. For every order, there is a net revenue of this order.
3. For every order, we need to calculate the monthly revenue. The definition of monthly revenue is `net revenue * the revenue days of certain month / total days`

If an order starts at 5/5, ends at 6/8 with total revenue as 35 dollars, then the total days of this order is (27+8) = 35 days. Also, the monthly revenue of May is 27 dollars and monthly revenue of June is 8 dollars.

To solve this, at the beginning, I used `first-day-of-the-month` and `last-day-of-the-month` in `clj-time` library to calculate how many days within a month. The first version solution was a traditional imperative solution. I quickly found that I could do better with functional thinking.

My improved version:
1. Generate a sequence of time using `period-sec` in `clj-time`. The period of time is just one day long and the start date/end date are the start date/end date of certain order.
2. Apply `group-by` to the step 1 day sequence with the grouping function that can return the year-month-string of a certain date. For example, a date of 2009/05/01 returns "2019-05".
3. Calculate how many days of each group of the step 2 result.
4. Spread the revenue using step 3 result.

## LambdaCD and ansible

I was not an expert of DevOps. When I needed to deploy the project, I took some time to study ansible because the great book `Deploying Your First Clojure App ...From the Shadows shows` introduced ansible. I still felt ansible is a great tool worth learning, however, the target servers were under the bastion host.

Engineers in LINE Taiwan told that they installed a Drone CI/CD server in the virtual private network under the bastion host. As a Clojure developer, I decided to use LambdaCD. Actually, it was even simpler than Drone. Parentheses abundant lisp clj files were more expressive than yaml.

When I encountered problems, I used github issues to ask at LambdaCD github repo. Within two days, the author of LambdaCD kindly replied my questions. I thought LambdaCD is worth of recommendation.

## Evangelism of Clojure

Given that I did software consulting at a big company, I applied for technical talk inside the company. Grabbing the chance, I introduced Clojure to 10~ developers. Those who already had experience with Scala showed more interests than others. Good beginning anyways, I thought. Here is the [slide](https://www.slideshare.net/humorless/the-productivity-brought-by-clojure-149170292/) of technical talk.

