---
layout: post
title: Required ports for WhatsApp
---

# Intro

Ran into a little fun time at work recently. A user was intermittenytly seeing WhatsApp fail to connect. Poked around for a bit and figured out that we didn't have all the required ports open on our guest wifi network. I spent a while trying to track down a definitive list of ports that are necessary for WhatsApp. Found some dated and inaccurate references scattered around, but finally got a true list from their support team. Enjoy!

# Ports

| Protocol | Port Number |
|----------|-------------|
| TCP      | 80          |
| TCP      | 443         |
| TCP      | 5222        |
| UDP      | 3478        |
| UDP      | 53          |
| UDP      | 40020       |
| UDP      | 57923       |