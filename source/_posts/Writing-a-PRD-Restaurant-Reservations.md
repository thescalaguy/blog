---
title: Writing a PRD - Curated Menus at Restaurants
tags:
  - misc
date: 2025-10-17 19:17:27
---


One of my favorite food delivery apps recently launched a feature where they spotlight restaurants. Upon clicking the icon which shows you more about the restaurant, you're presented with a list of all the outlets they have in the city. As someone who eats outside frequently and eats a variety of different cuisines, this got me thinking. What if we could elevate this experience? While previously I'd written about a feature I'd love to see from a programmer's perspective, I'll write this post from a product manager's perspective. In essence, I'll attempt to write a Product Requirements Document (PRD). I'll follow the ["How to write a PRD"](https://www.notion.com/blog/how-to-write-a-prd) guide by Notion. Here it goes.

## Step 1 - Define the Product. 

The aim of the product is to elevate a diner's experience of a restaurant by presenting them with curated menus. Curated menus allow the diners to expreience the best of what the restaurants have to offer by letting them focus more on what makes eating out an experience -- the cuisine, the company, and the conversations.   

There are various personas who will benefit from such a product. These personas are defined below.  

### Persona 1 -- The Explorer. 

People who like eating out frequently and trying new places and cuisines to eat can benefit from handpicked dishes presented in the curated menu. This allows them to experience new tastes and cultures whle enjoying a tailored exprience; every dish in the menu is chosen to delight the diner by highlighting the best of what the culture has to offer.  

### Persona 2 -- The Executive.  

Curated menus also make dining in a business setting easier by letting the diners focus more on enjoying the cuisine, and having business discussions than choosing dishes out of a menu. This also allows highlighting, if there are diners from various countries, the cuisine of the host country.

### Persona 3 -- The Lover.  

Curated menus make going out on dates a more memorable experience by allowing partners to bond over meals shared together. The menu takes the mystery out of choosing the right dish as each is chosen to delight the diner. This makes going out and choosing the right restaurant for the evening an easier experience.

## Step 2 - Determine Goals.  

The following are the list of SMART goals which will enable the release of the first iteration of the product.

1. Identify the top three cities where users eat out the most.  
2. Identify the five restaurants in these cities who'd participate.  
3. Create three curated menus for each of these restaurants.
4. Get 100 bookings for each menu in the first six months.  

The KPIs for the following goals are listed below.  

1. The number of people viewing curated menus.  
2. The number of people buying curated menus.
3. The number of people adding curated menus to their favorites.

## Step 3 -- Constraints and Limitations.  

Curated menus take time to create. Getting each restaurant onboard, creating the menu, presenting them to the user, and getting them to choose the menu can be a time-taking process. Additionally, factoring in people's dietary choices mean that some menus will have limited appeal thus reducing the number of people who choose the menu. Another factor is the pricing of the menu requiring them to be created at different price points.  

## Step 4 -- Scope.  

The initial pilot of the feature should include only a limited number of restaurants and menus. From the point-of-view of functionality, the engineering effort required should focus on enabling the foals outlined in Step 2 -- adding curated menus in the system, displaying them to users, allowing users to buy the menu, notifying the restaurants of the purchase, and analytics to track the KPIs.

## Step 5 -- Features. 

{% asset_img App.png %}  

Upon viewing a restaurant, the user will be presented with the list of curated menus for that restaurant along with the price. They will be able to buy it within the app by choosing a date and time, and paying the displayed amount. This will reserve a table for them and notify the restaurant that a curated experience is expected at the chosen time.  

## Step 6 -- Release Criteria.  

The criteria for success would include trial runs with restaurants with a chosen group of users and ensuring that the functionality performs as expected. Once past this phase, the feature can be released generally to more users. Concretly, the following criterias would help check the readiness of the product.  

1. Add the participating restaurants into the system.  
2. Add the curated menus into the system.  
3. Ensure that the menus show up in the app as expected.  
4. Purchase a menu and check the end-to-end flow.  
5. Release it to an early group of users.  

Finito.

