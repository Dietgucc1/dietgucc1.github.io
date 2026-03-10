---
title: "MALICIOUS NETWORK TRAFFIC DETECTION AT DARK WEB: A PROOF OF CONCEPT"
date: 2026-03-10 00:00:00 +0800
categories: [FYP, Cybersecurity]
tags: [fyp, machine-learning, dark-web, network-traffic, random-forest, tor, vpn, i2p]
image:
  path: /assets/img/riskweightdiagram.png
  alt: Malicious Network Traffic Detection Model for Dark Web
description: How I built a machine learning model that detects dark web traffic without decrypting a single packet — achieving 98.2% accuracy as a proof of concept.
---
# MALICIOUS NETWORK TRAFFIC DETECTION AT DARK WEB: A PROOF OF CONCEPT — My Final Year Project

*By Nur Adlina Auji Binti Mohammed Zulkarnain | December 2025*

---

## The Problem Nobody Talks About

When we think about cybersecurity threats, we often imagine hackers breaking into systems. But there's a quieter, harder problem — **what happens when the criminals are the ones using encryption correctly?**

The dark web runs on anonymity. Tools like Tor, I2P, Freenet, and ZeroNet wrap network traffic in layers of encryption so strong that traditional monitoring methods simply cannot see inside. You can't inspect the payload. You can't read the content. So how do you detect malicious activity?

That question became the foundation of my Final Year Project (FYP) 
---

## What I Built

My project, titled **"Malicious Network Traffic Detection Model for Dark Web: A Proof of Concept"**, proposes a machine learning framework that can detect and classify dark web traffic — **without ever decrypting a single packet**.

Instead of looking *inside* the traffic, I trained models to look at *how* the traffic behaves. Think of it like identifying a person not by reading their diary, but by observing their walking pattern, their schedule, and the doors they knock on.

### The Two-Phase Approach

The system works in two stages:

**Phase 1 — Encrypted or Not?**
The model first determines whether a network flow is encrypted or plain traffic. This filters out irrelevant data early and reduces noise.

**Phase 2 — Which Darknet Protocol?**
Encrypted traffic is then classified into one of five anonymisation technologies:
- Tor
- VPN
- I2P
- ZeroNet
- Freenet

This layered design means the system doesn't waste resources analysing non-encrypted traffic, and it dramatically reduces false positives.

---

## The Results

Four machine learning algorithms were tested: Logistic Regression, Support Vector Machine (SVM), Convolutional Neural Network (CNN), and Random Forest.

**Random Forest came out on top — consistently.**

| Phase | Model | Accuracy |
|---|---|---|
| Phase 1 (Encrypted vs Non-Encrypted) | Random Forest | **98.2%** |
| Phase 2 (Multi-class Darknet) | Random Forest | **90.4%** |

These results were validated using 10-fold cross-validation on the **BCCC-Darknet-2025** dataset — a real-world network traffic dataset containing actual traffic from Tor, VPN, I2P, ZeroNet, and Freenet.

![Phase 1 Model Comparison Graph](/assets/img/figure4.1.png)
*Figure 4.1: Phase 1 model performance comparison — Random Forest leads across all metrics*

![Phase 1 Evaluation Metrics](/assets/img/figure4.1(2).png)
*Figure 4.1 (detailed): Stage 1 evaluation metrics across all four models*

![Phase 2 Classification Performance](/assets/img/figure4.2.png)
*Figure 4.2: Phase 2 encrypted traffic classification performance across models*

![Phase 2 Evaluation Metrics](/assets/img/figure4.2(2).png)
*Figure 4.2 (detailed): Stage 2 evaluation metrics for multi-class darknet classification*

---

## The Most Interesting Finding

When I ran feature importance analysis to understand *what* the model was actually learning, the results were genuinely surprising.

### For detecting encryption (Phase 1):
**TCP congestion signalling flags — specifically CWR and ECE — accounted for 56.25% of the model's decision-making power.**

These are small flags in the TCP protocol that signal network congestion. Encrypted sessions use them in a distinct, stable pattern that non-encrypted traffic doesn't replicate. Nobody told the model this. It figured it out on its own.

![Top 16 Features Phase 1](/assets/img/figure4.3.png)
*Figure 4.3: Top 16 features for encryption detection — TCP congestion flags (CWR & ECE) dominate at 56.25%*

### For identifying specific darknet protocols (Phase 2):
**Packet inter-arrival time (IAT) and directional flow statistics provided over 80% of the discriminative power** for identifying Tor and I2P traffic.

In a simple word: each anonymisation network has a unique "rhythm." Tor's multi-hop relay routing creates irregular timing gaps. I2P's tunnel architecture produces stable, predictable timing patterns. VPN tunnels have symmetric, continuous flows. The model learns these rhythms without ever seeing what's inside the packets.

---

## Why This Matters

### Privacy-Preserving by Design
Traditional deep packet inspection (DPI) requires reading packet content — which raises serious legal and ethical concerns, especially in environments where privacy laws like Malaysia's **PDPA 2010** and the EU's **GDPR** restrict surveillance. My model works entirely at the *flow level*, meaning it never touches payload data. It's built to be legally compliant from the ground up.

### Real-World Relevance
Dark web threats are not abstract. In January 2024, Malaysian police arrested an individual for leaking 17 million MyKad records on the dark web. In March 2025, Singapore authorities seized 154 replica firearms purchased through dark web channels. These incidents highlight why tools that can detect dark web activity — without violating privacy — are urgently needed.

### Risk-Weighted Intelligence
The system also assigns a relative risk weight to each detected traffic type, helping analysts **prioritise** their response:

![Risk Weight Mapping](/assets/img/riskweightdiagram.png)
*Figure: Relative illegal online trade risk mapping — from VPN (lowest) to Tor (highest)*

This isn't about criminalising VPN usage. It's about giving cybersecurity analysts the context they need to make smarter decisions.

---

## What I Learned

Building this project taught me that **you don't always need more data — you need the right features.** By selecting only the top 16 most discriminative features from five functional categories (Basic Flow Statistics, Statistical Measures, Protocol-Specific Features, Advanced Metrics, and Directional Analysis), the model remained lightweight, fast, and interpretable — qualities that matter enormously in real-world deployment.

It also reinforced something important: **machine learning is a tool, not magic.** Understanding *why* a model makes a decision — through feature importance analysis and explainability — is just as valuable as the accuracy score itself.

---

## What's Next

This is a proof of concept, and there's a lot of room to grow:

- Extending the model to handle **real-time traffic capture**
- Exploring **transformer-based architectures** for sequential traffic modelling
- Adding more anonymisation technologies as the dark web ecosystem evolves
- Integrating with **live network monitoring and incident response systems**

---

## Final Thoughts

The dark web isn't going away. Encryption isn't going away either nor should it. The challenge for cybersecurity is learning to work *alongside* privacy, not against it.

This project is my small contribution to that challenge: a system that can see patterns in the dark, without needing to turn on the lights.

---

*This research was completed under the supervision of Prof. Ts. Dr. Madihah Mohd Saudi at the Faculty of Science and Technology, Universiti Sains Islam Malaysia (USIM), December 2025.*

*Dataset used: BCCC-Darknet-2025 | Tools: Python 3.12, Scikit-learn, TensorFlow (Keras), Pandas, NumPy, Matplotlib*
