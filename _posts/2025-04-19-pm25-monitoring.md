---
layout: post
title: "Live Air Pollution Monitoring"
date: 2025-04-19
lang: en    
---

The plot below shows real-time PM2.5 levels measured in my area. This page updates with fresh data several times per hour.

<iframe src="/assets/pm25_plot.html" width="100%" height="650" style="border:none;"></iframe>

## Live Air Pollution Monitoring

I live close to Copenhagen Airport, and I’ve been curious — and increasingly concerned — about the local air quality, particularly due to particle emissions from aircraft. This project is my attempt to monitor the environment right outside my door.

According to the Danish Ministry of the Environment, an estimated **4,000 Danes die prematurely every year** due to air pollution--that's more than 10 people every day.

> [Ny rapport: Bedre luftkvalitet medfører 380 færre for tidlige dødsfald pga. luftforurening (Miljøministeriet, marts 2025)](https://mim.dk/nyheder/pressemeddelelser/2025/marts/ny-rapport-bedre-luftkvalitet-medfoerer-380-faerre-for-tidlige-doedsfald-pga-luftforurening)

**Air pollution has serious consequences** — it increases the risk of asthma, lung cancer, heart disease, and strokes. And it’s not just a problem in big cities. Even suburban areas near airports or highways can have elevated levels of particulate matter.

---

## What is PM2.5?

**PM2.5** refers to fine particulate matter that is 2.5 micrometers or smaller in diameter. These particles are small enough to penetrate deep into your lungs and even enter your bloodstream. Sources include vehicle exhaust, industrial processes — and notably, aircraft emissions.

---

## How It Works

This is a fully DIY solution built with open-source tools and low-cost hardware:

- I'm using an **ESP32** microcontroller connected to a **PMS5003** particle sensor.
- The ESP32 pushes data to a **bucket on InfluxDB Cloud** every few minutes.
- A **cron job on my Raspberry Pi** pulls this data, processes it with a Python script, and uploads it to a **public GitHub repository**.
- This blog then loads the latest CSV data and renders it using **Plotly.js** in your browser.

Simple. Cheap. Open.

---

## Future Plans

I'm currently working on a **custom PCB** to integrate everything into a more compact and robust form factor:

- It will include the PMS5003, an ESP32, and support for a small LCD display.
- I'm also designing a **3D-printed enclosure** to make the sensor weatherproof and mountable outdoors.
- Here's a preview of the prototype being manufactured:

![PCB Prototype]({{ site.baseurl }}/assets/images/pcbway.jpg)

I designed the board in **KiCAD**, and it's currently being fabricated by **PCBWay**.

---

## Photos of the Current Setup

Here’s the current test setup, mounted in my garden:

![Open Weatherstation]({{ site.baseurl }}/assets/images/open_weatherstation.jpg)

And here’s the final version enclosed and deployed:

![Weatherstation]({{ site.baseurl }}/assets/images/weatherstation.jpg)

---

**Stay tuned!** I’ll be posting updates as the hardware evolves and more sensors are added — maybe even CO₂ and NOx in the future.

*– Anders*

