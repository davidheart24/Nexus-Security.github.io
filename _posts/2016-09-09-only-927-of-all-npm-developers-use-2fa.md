---
title: 'Only 9.27% of all npm developers use 2FA'
date: 2020-01-07T05:38:00+01:00
draft: false
---

![npm](https://zdnet2.cbsistatic.com/hub/i/2019/10/15/54d57679-a8a4-483a-930c-dc701f67334a/npm.png)

Image: npm

Only 9.27% of all maintainers of npm JavaScript libraries use two-factor authentication to protect their accounts.

The number is incredibly low and a major issue of concern for the npm security team, who'd like to see this figure grow in the coming year.

[Npm](https://www.npmjs.com/package/repository), which stands for Node Package Manager, is one of the many package (library) management systems for the JavaScript ecosystem.

Npm is a company, a web portal listing all JavaScript libraries, and a command-line tool for importing those libraries in a JavaSript project, may it be a desktop, mobile, web, or server-side app.

Today, npm is, by far, the largest JavaScript package manager in the JavaScript ecosystem, but also the largest package repository for any programming language, [with more than 350,000 indexed libraries](https://developers.slashdot.org/story/17/01/14/0222245/nodejss-npm-is-now-the-largest-package-registry-in-the-world).

This has made npm a prime target for supply-chain attacks, scenarios where hackers breach a developer's npm account in order to insert malicious code inside their libraries. Such incidents have happened in the past years, including 2019.

*   [June 2019](https://www.zdnet.com/article/cryptocurrency-startup-hacks-itself-before-hacker-gets-a-chance-to-steal-users-funds/) - a hacker backedoored the electron-native-notify library to insert malicious code that reached the Agama cryptocurrency wallet.
*   [November 2018](https://www.zdnet.com/article/hacker-backdoors-popular-javascript-library-to-steal-bitcoin-funds/) - a hacker backdoored the event-stream npm package to load malicious code inside the BitPay Copay desktop and mobile wallet apps, and steal cryptocurrency.
*   [July 2018](https://eslint.org/blog/2018/07/postmortem-for-malicious-package-publishes) - a hacker compromised the ESLint library with malicious code that was designed to steal the npm credentials of other developers.
*   [May 2018](https://blog.npmjs.org/post/173526807575/reported-malicious-module-getcookies) - a hacker tried to hide a backdoor in a popular npm package named getcookies.

Academic research published last year showed that most of the npm packages are intertwined with one another, and that [hacking 20 high-profile developer accounts](https://www.zdnet.com/article/hacking-20-high-profile-dev-accounts-could-compromise-half-of-the-npm-ecosystem/) could allow a threat actor to plant malicious code that gets used by half of the entire npm ecosystem.

As such, securing the accounts of npm library owners should be a top priority going forward.

The 9.27% figure is pretty low, and the npm team should take a page out of Mozilla's book, the company behind the Firefox browser.

Last month, Mozilla announced that starting with January 2020, [all developers of Firefox browser extensions must enable 2FA](https://www.zdnet.com/article/mozilla-to-force-all-add-on-devs-to-use-2fa-to-prevent-supply-chain-attacks/) for their accounts in order to update their extensions going forward.

Other security-related stats from the npm security team \[[source](https://twitter.com/adam_baldwin/status/1212158657451851778)\]:

*   Number of npm tokens revoked erroneously published to either the registry or to GitHub: 737
*   Total security advisories in the npm database: 1,285
*   Security advisories created in 2019: 595
*   Percentage of new account passwords improved by rejecting reused passwords compromised in previous breaches: 13.37
*   Number, in millions, of run-time reports generated by our behavioral analysis API: 1.4

  
  
from Latest Topic for ZDNet in... https://ift.tt/36z4pAY