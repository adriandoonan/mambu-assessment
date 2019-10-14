## mambu technical writer assessment

### challenge

> As the last step of the interview process, we would like you to rewrite/restructure the following user guide article: https://support.mambu.com/docs/card-payments-and-authorization-holds. The format is entirely up to you and does not need to resemble existing Mambu documentation at all. Please return the assignment to us by Monday, Oct. 14. Feel free to reach out to Dustin with any questions.

### published site

The site is currently published at https://adriandoonan.github.io/

### initial thoughts

- quite a bit of background reading was needed to get up to speed on the banking industry jargon
- wasn't clear whether "advice" simply meant adding "advice=true" to api requests
- some parts reference functionality that does not seem to be documented and some features referenced in the API documentation is not mentioned here (eg. delete request for reversing a hold)
- naming was sometimes different between this documentation and api documentation (eg. request vs create authorization hold)

### changes

- separated operations into those following the authorization hold flow and other types of transactions
- included a few more real-world use-case examples
- tried to make the requirements a bit more visible
- tried to reorder the sections in a more logical way going from the request -> implications for account balance -> what will happen if no actions are taken -> actions which can be taken
- split section for reversal into two (reversing holds and refunding already settled transactions)
- included more links to the api documentation

### final thoughts

- I underestimated the task a little and spent more time than expected in a state of mild confusion once I really started to dig in
- things got easier once more research had been on the topic
- I should not have left it to the weekend as that removed the possibility to ask follow up questions for sections I did not fully understand
