---
date: "2025-04-03T14:00:54-08:00"
title: "Prompt Injection Exploits on Browser Use"
authors: ['michaelye', 'colin', 'zimo']
categories:
 - AI
tags:
 - Python
draft: false  
---
# Prompt Injection Exploits on Browser Use Agent


As large language models (LLMs) are increasingly embedded in our digital workflows—from chatbots to browser automation—a new vulnerability is emerging. Browser agents can now perform tasks such as reading web pages, clicking links, and even handling sensitive data with human-like precision. But with great power comes great responsibility, and our recent research reveals that these agents are susceptible to prompt injection attacks that could lead to severe security breaches.


---


## Introduction


Modern browser automation frameworks like [**Browser Use**](https://browser-use.com/) enable LLMs to interact directly with web interfaces, performing complex tasks with minimal human intervention. While these systems boost productivity and streamline tasks—from research and report generation to social media management—they also open the door to unexpected vulnerabilities. Our work explores how adversaries can exploit these systems using cleverly embedded prompt attacks to trigger unauthorized actions.


---




## How Browser Agents Work


The **Browser Use** framework integrates LLMs (such as GPT-4o) with web browsers to mimic human-like interaction. The agent can:
- Navigate to websites and interact with web elements.
- Extract and process both textual and visual data.
- Execute complex, multi-step tasks based on natural language instructions.


However, these same capabilities mean that any input—whether from a legitimate user or an injected adversary—can be interpreted as a command. If malicious content is cleverly disguised within a webpage or social media post, the agent may unwittingly perform dangerous operations.


---


## The Anatomy of a Prompt Injection Attack


Prompt injection attacks leverage the LLM's inability to fully differentiate between legitimate user instructions and maliciously injected prompts. 


### Threat Model


- Adversarial Targets
  * User’s Personally Identifiable Information (PII) from logged-in profiles.
  * Email contents and data stored in the cloud (Google Drive).
- Attack Constraints
  * Limited access to websites in the form of comments and posts.
  * Not necessarily invisible to humans given minimal human supervision.
  * Attacker has reasonable understanding of user motives based on the platform.
- Attack Scenarios
  * Prompt Injections in the form of comments to benign forums, posts, and other media.
  * Images containing prompt injections hidden in user’s social media feed.




Our experiments focused on two main types:


### Text-Based Injections


- **Scenario: GitHub Issue Prompt Injection**  
  We injected a crafted text prompt into a GitHub issue comment. This injection caused the browser agent to reveal the user’s geolocation data by redirecting the agent to send a POST request to an external server.


- **Scenario: Google Docs Comments Injection**  
  By embedding malicious text within YouTube comment sections, the agent was tricked into accessing and forwarding sensitive email data.
  ![](../../static/videos/google_docs_attack.mp4)


### Image-Based Injections


- **Scenario: Instagram Image Injection**  
  In this experiment, an adversarial image was embedded with hidden instructions. When the agent processed the image, it altered file access permissions on a Google Drive folder and automatically emailed the public link to an attacker-controlled address.  


The experiments highlighted that, regardless of the medium (text or image), once the malicious prompt is merged with the agent’s original workflow, the LLM’s safeguards are bypassed, and the agent follows the unintended command.


---
### Leaking User Information Across Multiple Platforms


Our attacks were not confined to a single website. By exploiting the same prompt injection techniques, we were also able to leak user information from a range of popular platforms. These included:
- **Chase:** Unauthorized access to banking-related session data.
- **Amazon:** Extraction of user account details and order history.
- **Leetcode:** Exposure of user profiles and problem-solving data.
- **Gmail:** Breaching sensitive email content.


Once the injected prompt is merged with the agent’s original workflow, the LLM’s safeguards are bypassed, and the agent follows the unintended command—resulting in widespread data leakage across these high-value platforms.
---


## Mitigation Strategies


Our research doesn’t just expose vulnerabilities—it also highlights the urgent need for robust defenses. We propose a multi-layered security strategy:


- **Web-Level Defenses:**  
  Implement stricter access controls (e.g., two-step authentication) for browser agents handling sensitive data. This helps to reduce the impact even if an agent is compromised.


- **Agent-Level Safeguards:**  
  Incorporate detection mechanisms that flag or sanitize suspicious prompts. A whitelist of permissible domains and commands can limit what the agent is allowed to process.


- **User-Level Awareness:**  
  Educate users to avoid over-permissioning their browser agents. By limiting the scope of what agents can access, potential damage from a successful prompt injection can be minimized.


---


## Why This Matters


The implications of our findings are significant. Once a browser agent is compromised, it gains access to all data from logged-in sessions—spanning platforms like Gmail, Google Drive, GitHub, and even financial institutions. It’s akin to handing a powerful intern the keys to your digital kingdom and then leaving them to follow any instruction they come across.


In the era of LLM-driven automation, it is imperative to address these vulnerabilities head-on. Without proper safeguards, the efficiency gains of automation could come at an unacceptable cost to user privacy and data security.


---


## Conclusion


Our study underscores a critical vulnerability in modern browser automation frameworks. Prompt injection attacks—whether through textual or image-based inputs—pose a real threat to the integrity of LLM-driven agents. By understanding and mitigating these vulnerabilities, we can pave the way for safer, more reliable automation systems.


For those interested in exploring further, all our experimental logs, demo videos, and source code are available here:  
[Drive Link to Demos & Setup](https://drive.google.com/drive/folders/1Z-Jon0766vPspmbOXw2_XubDBlZ9TVwX?usp=sharing)


---


## Credits


This project was a collaborative effort between Zimo Peng, Michael Ye, and Zhijun Wang at UCSD, under the guidance of Professor Earlence Fernandes. Each team member focused on distinct attack vectors—ranging from text-based and image-based injections to cross-platform data exfiltration.


---


## TL;DR


> Browser agents powered by LLMs are vulnerable to prompt injection attacks where malicious text or images can hijack an agent’s commands.  
> Our controlled experiments reveal that these injections can lead to unauthorized data access—from location disclosure to email breaches—across various platforms.  
> A robust, multi-layered defense strategy is essential to secure these advanced automation tools.