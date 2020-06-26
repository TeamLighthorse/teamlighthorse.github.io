---
layout: post
title: App Insights
tags: [lighthorse, App Insihgts]
excerpt_separator: <!--more-->
author: matthews58
---

This post will teach you about Application Insights for web pages specifically for Angular.

<!--more-->


### What is Application Insights?

Application Insights is an application performance management service for web applications. With application insights, you can do all the monitoring of your website performance in Azure. It includes tools that help you diagnose issues and understand what users actually do with your app. It's designed to help you continuously improve performance and usability.

### What are some things Application Insights can monitor?

- **Request rates, response times, and failure rates** - find out which pages are most popular, at what times of the day, and where your users are.
- **Exceptions** - Analyze the aggregates statistics, or pick specific instances and drill into the stack trace and related requests. Both server and browser exceptions are reported.
- **Page views and load performance** - reported by your users' browsers.
- **AJAX call from web pages** - rates, response time, and failure rates.
- **User and session counts**
- **Custom events and metrics** that you write yourself in the client or serve code to track business events.


### How to add Application Insights to Angular App?

1. Add Application Insights to Azure
2. Login to [Azure Portal](https://portal.azure.com/)
3. Since we already have our resources created, go to:
    - Resource Groups
    - Lighthouse-Core-DEV (for Development)
    - lighthouse-tms-dev-appinsights
    - Copy the Instrumentation Key
4. Configure Angular App by adding your instrumentation key
    - For our project we have added our instrumentation keys to **each** environment. For example: in appsettings.Development.json we added: 
```json   
    "ApplicationInsights": {
	"InstrumentationKey": "<Your instrumentation key>"
}
```
5. Install Application Insights Javascript SDK: `npm install @microsoft/applicationinsights-web`
6. Now we will create a service to communicate with azure application insights:

```ts
import { Injectable } from '@angular/core';
import { Store } from '@ngrx/store';
import { currentUserInfo } from '../account/store';
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { Router, NavigationEnd } from '@angular/router';
import { filter } from 'rxjs/operators';

@Injectable({
	providedIn: 'root'
})
export class AppInsightsService {
	private AppInsights: ApplicationInsights;

	constructor(
		private store: Store<any>,
		private router: Router
		) {
			this.configureAppInsights();
		}

	logPageView(
		name?: string,
		uri?: string
	) {
		this.AppInsights.trackPageView({name, uri});
	}

	logEvent(
		name: string,
		properties?: any,
	) {
		this.AppInsights.trackEvent({name, properties: {properties}});
	}

	logException(
		exception: Error,
		properties?: any,
		severityLevel?: any
	) {
		this.AppInsights.trackException({exception, severityLevel, properties: {properties}});
	}

	startTrackEvent(
		name: string
	) {
		this.AppInsights.startTrackEvent(name);
	}

	stopTrackEvent(
		name: string,
		properties?: any,
	) {
		this.AppInsights.stopTrackEvent(name, properties);
	}

	startTrackPage(
		name?: string
	) {
		this.AppInsights.startTrackPage(name);
	}

	stopTrackPage(
		name?: string,
		url?: string
	) {
		this.AppInsights.stopTrackPage(name, url);
	}

	private createRouterSubscription() {
	  this.router.events.pipe(filter(event => event instanceof NavigationEnd))
	  	.subscribe((event: NavigationEnd) => {
			this.logPageView(event.url, event.urlAfterRedirects);
	  	});
	}

	configureAppInsights() {
		this.store.select(currentUserInfo)
			.subscribe(userInfo => {
				this.AppInsights = new ApplicationInsights({
					config: {
						instrumentationKey: userInfo.appInsightsKey,
						maxAjaxCallsPerView: 10
					}
				});
			this.AppInsights.loadAppInsights();
			this.AppInsights.setAuthenticatedUserContext(userInfo.username, userInfo['selectedCustomerCode']);
			this.createRouterSubscription();
		});
	}
}
```

 - Here you can include however many methods in [the SDK](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md) that you want.
 - For us, we are logging pageViews (start and stop page views), events (start and stop events), and exceptions.
 - createRouterSubscription() listens for navigation events and then logs these as pageViews. Since we are using Angular router, we need this because Application Insights only logs full page refreshes by default.
 - configureAppInsights() goes out and gets the current user information and sets the instrumentationKey. It then calls loadAppInsights() which initializes the instance of ApplicationInsights. Next, it calls setAuthenticatedUserContext which will set the authenticated user id and the account id. Finally, it will call createRouterSubscription() to start listening to navigation events.


### Azure Portal

1. Login to [Azure Portal](https://portal.azure.com/)
2. Go into the resource group you want to monitor for example
	- Resource Group: Lighthouse-Core-DEV (for Development)
    - Application Insights: lighthouse-tms-dev-appinsights
3. Go into the Search tab under Investigate and change event types to Page View
	- Here you can see all of the pages that have been viewed and their page view load times
<kbd>
  <img src="/assets/img/logging2.png">
</kbd>

4. Change the event types to Custom Event and choose a result
	- Now you can see all custom events along with their custom properties such as duration and status
<kbd>
  <img src="/assets/img/logging7.png">
</kbd>

5. In the side menu, go to Failures
	- You can see any operations that have failed on the server side
<kbd>
  <img src="/assets/img/logging9.png">
</kbd>

6. If you switch from Server to Browser and select the Exceptions tab you can see console errors (if there are any) that occured in the browser
<kbd>
  <img src="/assets/img/logging8.png">
</kbd>