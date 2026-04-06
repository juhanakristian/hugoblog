---
title: "Why you should use the URL to store state in your application"
draft: false
date: "2021-11-28T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["webdev"]
description: "It seems we have, in the web community, collectively forgotten the role of the URL in storing the state of our application."
---

It seems we have, in the web community, collectively forgotten the role of the URL in storing the state of our application. For good or bad, we've started developing websites like they are apps and forgotten one of the key elements of the web. I've seen (and built) applications where opening modals, searching, filtering, or navigating a wizard doesn't change the URL. And this is wrong because it doesn't fit the web paradigm.

Websites work in the context of the browser, and can never control the user flow completely as desktop or mobile applications can. At any point, the user can hit refresh, navigate back/forward or close the tab. These are all things we, as web developers, need to take into account. We should **take ownership** of these interactions and design our applications so these actions result in an outcome the user expects!

How can we do better? We need to think about the actions our users take, and the expectations they have when using our application. What should happen when you hit refresh? Or navigate back? What state do we need to retain? What inputs need to be initialized from the URL?

### Search and filtering

![A list of invoices](/images/url-invoice-list.png)

Let's start with something clear-cut in my opinion. Search forms should be saved to the URL because the expectation is that you can copy-paste and share the URL and if you refresh the page, you will get the same search results.

How do you do it then? Well, this is what the [url search params](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) are for! Store each field of the search query in the URL and initialize the inputs from the URL when the page loads.

The same concept can be applied to filtering data in a view. For example, if we have a view with rows of invoices the user might want to filter out the invoices with which have been paid before a specified date (for example `/invoices?paidBefore=01-01-2021`). Now our users share this URL and the other user will see the same invoices.

### Detail views

![Detail view](/images/url-detail-view.png)

Another clear-cut case is detail view layouts. In a detail view layout, you have the main layout with items and a child layout with a detail view of the selected item. In this case, we can use the URL to store the id of the selected item.

For example, if we have a list of invoices at the URL `/invoices` and want to display the details when an invoice is clicked, we can have a link to `/invoices/1` and render the detail view when the URL has the id at the end.

### Modals

![Modals](/images/url-modal.png)

With modals, we can use the URL to store the **open** state. This is useful when the modal is used to add/edit entries or to show a detail view. With other modals, like alert/notification modals, it doesn't make sense because they are used to inform the user about some action they tried to perform.

When a user clicks a button or a link that opens the modal, the URL should change from for eg. `/invoices/` to `/invoices/add` and this should trigger the opening of the modal. On edit modals, we can use the id of the data were editing in the URL. So for example `/invoices/1/edit`.

This allows our users the share a link that opens the edit dialog of a specific invoice. Notice that while in the search interface we stored the input values in the URL, in other forms/modals we don't want to do this because the expectation is that if you cancel the modal or refresh the page, the data is lost. It is, however, a good idea to inform the user before a page reload about the possible loss of data.

### Wizards and stepped user interface

![Wizard](/images/url-wizard.png)

As with modals, there are many types of wizards or stepped interfaces we might have in our applications. The problem we face with wizards is that unless you use Local Storage or Session Storage to save the data, you will lose it on page refresh even if the position in the wizard is retained. The wizard position should therefore only be saved if we can also save all the data the user has entered in previous steps.

On some wizards, it might even make sense to store the state in the database. This would enable the user to come back to the wizard later, and continue the process where they left last time.

## Summary

The main benefits of this approach are a better **user experience** and **productivity**. When our users can share _deeper_ links they are more productive and less frustrated with our application. At the same time, the browser controls (refresh, navigate back/forward) result in an application state which the user can expect.
