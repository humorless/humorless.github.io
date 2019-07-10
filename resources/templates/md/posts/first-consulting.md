{
 :title "Lessons learned from the software consulting job"
 :layout :post
 :date "2019-06-23"
 :tags ["consulting", "experience"]
 :toc true
}

I live in Taiwan and I can not find Clojure jobs here. Although the first legal gay wedding in Asia took place here, it seems that the real programming language innovation still needs some evangelists to spread it. Therefore, I decide to create Clojure job by myself. In January this year, I had a chance to develop enterprise software for the LINE Taiwan, and I chose Clojure as my primary technical stack.

## Technical stack issues

When I discussed with my clients about this enterprise software solution, we focused on the problem domain. However, when I told my clients that I want to use Clojure, Datomic, and ClojureScript, my clients said no. They said a lot of cliches like they never hear Clojure before, it would be difficult to find Clojure programmers. Then, I made some compromises: I would use React with javascript in frontend but Clojure in backend with Datomic as database. For Clojure, I provided the reason that the business requirements had temporal queries which were like a piece of cake for Datomic but very time-consuming for traditional relational databases.

After developing this project for a while, I regretted that I did not insist on ClojureScript. I really spent a lot of time on javascript boilerplate code, and the time spent did not bring any value to my clients.

## A very simple user login is good enough for a small group of users

The enterprise software solution needed to be an on-premise solution, installed on the private network at LINE offices. There would be about 30 users login everyday. At the beginning, I thought three different ways to solve the user login problems:

1. Single signed-on with other enterprise software in LINE
2. Leverage third party authorization service
3. Traditional user login backend APIs and frontend UI with login/register/user management functions like resetting password.

Option 2 might be fast enough, but my clients did not like third party service.

My final proposal was a login module like this:
1. Frontend UI provided the login and password modification functions to ordinary users.
2. The administrator of this system used ETL (extract-transform-load) to manage user accounts. Given this design, we did not need any user registration or user accounts management UI.

## Revenue spreading problem

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

## CI/CD issues

I was not an expert of DevOps. When I needed to deploy the project, I took some time to study ansible because the great book `Deploying Your First Clojure App ...From the Shadows shows` introduced ansible. I still felt ansible is a great tool worth learning, however, the target servers were under the bastion host.

Engineers in LINE Taiwan told me that they installed a Drone CI/CD server in the virtual private network behind the bastion host. As a Clojure developer, I decided to use LambdaCD. Actually, it was even simpler than Drone. Parentheses abundant lisp clj files were more expressive than yaml files.

When I encountered problems, I asked questions at LambdaCD github repo. Within two days, the author of LambdaCD kindly replied my questions. I thought LambdaCD is worth of recommendation, both the quality of the software and quick response.

## Evangelism of Clojure

Given that I did software consulting at a big company, I could apply for technical talk inside the company. Grabbing the chance, I introduced Clojure to 10~ developers. Those who already had experience with Scala showed more interests than others. Good beginning anyways, I thought. Here is the [slide](https://www.slideshare.net/humorless/the-productivity-brought-by-clojure-149170292/) of technical talk.

