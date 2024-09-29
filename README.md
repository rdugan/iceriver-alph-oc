# IceRiver AL Series Overclocking Firmware
Modified firmware for all IceRiver Alephium/blake3 ASICs, adding customizable clock and voltage control, sensor graphing, properly secured login and API access, and other goodies.

Firmware files can be downloaded from the Releases section on the right hand side of this page.

If you have any issues, finding me (pbfarmer) in the Alephium Discord will probably result in the fastest response/resolution.

<br>

# Table of Contents
- [Fee](#fee)
- [Known Issues](#known-issues)
- [Features](#features)
  - [Configuration additions and updates](#configuration-additions-and-updates)
    - [Configurable clock and voltage offset](#configurable-clock-and-voltage-offset)
    - [Better fan controls](#better-fan-controls)
  - [Additional telemetry and other changes to home page](#additional-telemetry-and-other-changes-to-home-page)
    - [Graphing of chip metrics and longer term hashrates](#graphing-of-chip-metrics-and-longer-term-hashrates)
    - [Uptime and job rate on pool status](#uptime-and-job-rate-on-pool-status)
  - [API](#api)
  - [General UI improvements](#general-ui-improvements)
  - [Optional commercial Features](#optional-commercial-features)
    - [Multiple user management](#multiple-user-management)
    - [Hashrate splits](#hashrate-splits)
    - [Branding/Logo replacement](#brandinglogo-replacement)
  - [Stability and security improvements](#stability-and-security-improvements)
    - [Primary pool health monitor](#primary-pool-health-monitor)
    - [Fix for web server crashes](#fix-for-web-server-crashes)
    - [New auth/auth routines](#new-authauth-routines)
    - [TLS certificate management](#tls-certificate-management)
    - [Healthcheck loop](#healthcheck-loop)
- [Installation](#installation)
- [Usage Tips](#usage-tips)
  - [Power and metering](#power-and-metering)
  - [Cooling](#cooling)
  - [Tuning](#tuning)
  - [Reproducing results from other firmware](#reproducing-results-from-other-firmware)
- [Let's Talk About Hashrates](#lets-talk-about-hashrates)

<br>

# Fee
* 1% for the base version
* 1.33% for the commercial/hosting version.  This version includes additional features useful for larger deployments, such as multiple user management, hashrate splitting (e.g. for setting up hosting fees), and brand/logo replacement.  This fee will increase in future releases as more features are added (such as aggregate dashboards, auto-tuning, etc), but is expected to be capped at 2%.
* Fee traffic is directed to geo.pbmine.com, which is currently just an alias for Vipor.net.  If the system is unable to connect to the pool for the fee, OC will be removed and performance reduced until a connection is once again available.  A warning indicator will appear in the UI in such situations.

<br>

# Known Issues
* If you are unable to load the UI in your browser, or you get errors such as 'ASIC request failed', the most likely cause is browser caching.  This can often be resolved simply by holding 'Shift' while pressing the browser refresh button.  Another simple way to test whether caching is the problem, is by opening the page in a 'private' or 'incognito' browser window.  If this works, then clearing your cache or site data should resolve the issue.
* IceRiver ASICs seem to insist on using static IP settings, even after you set them to DHCP.  If these IPs are not reserved in your router, at some point your router may assign these IPs to another device in your network, and you will have network / connection problems.  The solution to this is to find the MAC address to IP address mapping table in the LAN/DHCP section of your router settings, and add a mapping for every one of your ASICs (the MAC address should be on a sticker on the outside of your ASIC.)  Once you've added these mappings, restart the router, update every ASIC to use DHCP, and restart all of the ASICs.  Now your ASICs should acquire the fixed IP via DHCP, and even if/when they revert to static IPs, it should no longer cause issues, since the router has reserved the IP(s).
* The fee traffic can possibly trip spam/ddos/botnet protections in their routers, which commonly target mining traffic.  If your ASIC seems to connect to pools but is not mining (even after following the previous steps), check for these types of settings in your router and try disabling them.  For example, in my ASUS router, I need to disable the 'AIProtection' features called 'Two-Way IPS' and 'Infected Device Prevention and Blocking' to do any sort of crypto mining.
* There may be incompatiblities between HiveOS/AsicHub or other 3rd party management/monitoring tools and this firmware.  Many of these tools use unauthenticated API calls previously exposed by the stock UI to get data.  All web pages and API calls are now behind proper auth/auth, and some of the API signatures have slightly changed as well.  The only way to address this is for the management tools to migrate to the available API.
* This firmware uses https, while the stock firmware uses insecure http.  Many (most?) browsers seem to have a problem reverting to http after having loaded a page via https.  In chrome, I often need to use incognito windows to access the ASIC after reverting to stock from this firmware.  Clearing site data and/or restarting your browser or pc may help if you're experiencing this.

<br>

# Features

## Configuration additions and updates

<br>

### Configurable clock and voltage offset

![Performance Settings](/docs/images/iceriver-oc_settings.png)

Clock and voltage offsets have been added to the 'Miner' page.  Clock can be increased/decreased to any integer value (within hardware limits - currently limited to -125mHz - +875mHz).  Clocks will be rounded down internally to the nearest multiple of 5mHz.  Changes take effect immediately without restart, but note that clock increases are gradually applied in increments of 25Mhz per 30s.  As a result, it may take some time to get to full speed, possibly even ~10 minutes, depending on how large of an offset you choose.  

Voltage can also be increased/decreased to any integer value (within hardware limits), with changes taking effect immediately.  Voltage will be rounded down internally to the nearest multiple of 2mV.

<br>

*IMPORTANT: THERE ARE CURRENTLY NO GUARDRAILS, AND NO LIMITS ENFORCED BY THIS SOFTWARE ON EITHER CLOCKS OR VOLTAGE, SO USE WITH CARE.*

<br>

### Better fan controls

![Performance Settings](/docs/images/iceriver-oc_fan_settings.png)

A new fan mode has been added which automatically adjusts fan speed to maintain both max hash chip and board temperatures.  Temps are read every 10s and fan speed is adjusted as necessary.  

Please note, this setting does not guarantee the set temperature.  It may be exceeded by up to ~5C during startup or other dynamic periods, but it should stabilize at or near the requested temperature.

If you find the target temps are exceeded beyond your comfort during startup or other dynamic periods, you should increase the min fan speed.

Fixed fan speeds will also now be reapplied at startup, after a ~1-2m delay, though it is a one-time application.  This means that if the underlying IceRiver software decides to change the fan speed again for some reason, this mode will not re-apply your setting.  Consider using the 'Target Temp' mode with an appropriate min fan speed as an alternative.

<br>

<br>

## Additional telemetry and other changes to home page

<br>

### Graphing of chip metrics and longer term hashrates

![Home Page](/docs/images/iceriver-oc_home.png)

Two hours of graphing has been added for all chip metrics, with filters for summaries (per board min/max/avg), board, or all chips.  

80c chip temps appear to result in ideal hashrate performance.  No guidance has been provided by IceRiver as to safe chip temp limits, but their miner software will actually throttle clocks above 110C.  At least following general guidance from G/CPUs is probably prudent (e.g. >90C warning zone, >95C danger zone, >105C critical zone).  

Please note that real-time voltage will never match your setting - drivers under load experience voltage drop, meaning the running voltage will always be below your voltage setting, with more load causing a greater drop.

Non-chip 'board' temp graphs have been added, which includes board mounted intake and exhaust sensor temps, as well as power stage (driver) temps.  In summary mode, the max power stage temp is shown for each board, while in board mode, the max power stage temp is shown for each group/controller (PSG).  Max recommended operating temp is 125C according to the chip documentation, though it is probably wise to keep a healthy margin below this temp.  

Please be aware, that temperature is not the only consideration for healthy operation.  Power/current draw is also a concern, for which we don't currently have visibility or specifications.

Hashrate graphing (as well as the headline stats) now includes 30m and 2hr tracking, and also includes board level filtering.  

Mouseover tooltips have been synchronized across all graphs, to help with diagnosing issues / anomalies.

Instantaneous values are shown in the legend, and individual lines can be disabled/enabled by clicking on the labels.  Graph scales are no longer zero based, and adjust depending on which lines are displayed, meaning they are no longer artifically flattend by poor resolution, and you can actually see the variability in each measurement.

*Hopefully this helps clear up how variable 5m readings really are.*

<br>

### Uptime and job rate on pool status

![Home Page](/docs/images/iceriver-oc_overview_stats.png)

The uninterrupted uptime, and job issuance rate are added to the pool stats section.  Job rate is simply an additional health indicator of a pool connection - currently job rates for the Alephium network appear to be around 1.2s per job, or 0.83 jobs per second with a variation of roughly +/- 15%.

Multiple status indicators have been added to the pool section to help diagnose different network / pool issues.  A gray busy (spinning) icon indicates the asic is attempting to connect to the pool.  A green busy icon indicates a network connection, but no stratum connection yet. A yellow warning icon indicates a successful stratum connection, but no jobs have been received.

<br>

## API

A new rationalized API including all of the additional features from the UI is now avaiable over https (port 443).  Access is restricted by properly functioning authorization and authentication using a request header of the form "Authorization: Bearer \<TOKEN\>", with \<TOKEN\> being replaced by one of the tokens created via the 'Account' page in the UI.

Just as you would update the login password, PLEASE DELETE/REPLACE THE DEFAULT API TOKEN if you plan on exposing your machine publicly, as it is the same across all machines by default.

Full documentation is available in [json format](/docs/apidoc.json).

<br>

## General UI improvements
* Dark mode!
* Auto refresh controls
* Responsive design
* Numerous other css/js fixes vs stock firmware.  This is a constant work-in-progress.

<br>

## Optional commercial Features

A number of features have been added to a separate 'commercial' build intended for hosting or other large deployments.  These builds include a 'c' after the version number (e.g. *pbv082ALPHc_al0update.bgz*), and currently have an additional 0.33% fee (1.33% vs the standard 1%).

### Multiple user management

In addition to the standard primary/admin user, multiple users w/ differing access permissions can be added.  This allows for setups where, for example, the machine owner can be granted direct access to the machine, with permissions to view the main monitoring page and change pool configurations, while being restricted from changing network, fan or clock/voltage parameters.

<br>

### Hashrate splits

The hashpower of the ASIC can be split to multiple endpoints based on a configurable percentage, to allow setting up hosting fees.  The number of splits is not limited, but please keep in mind that the firmware will maintain a pool/stratum connection for each split, which multiplies the incoming traffic.

***This feature can also be used for splitting hashrate across multiple blake3 coins at once***

<br>

### Branding/Logo replacement

The 'PbFarmer' logo can be replaced with a branding image of your choice.  The image format should be a 112x60 PNG.

<br>

<br>

## Stability and security improvements

### Primary pool health monitor
Health-check loop run on primary pool availability.  If miner has switched to one of the secondary pools for any reason, you will be switched back to your primary pool as soon as it becomes available again.

<br>

### Fix for web server crashes
Replaced stock web server with updated and production environment targeted version, added cache/memory control configuration, and fixed memory leaks.  This should address the issues seen by users of HiveOS and other external monitoring tools that caused the web server to crash after too many page loads (resulting in the ASIC UI being unavailable.)

<br>

### New auth/auth routines
The authentication and authorization controls have been completely replaced, and all traffic redirected over https.  This means forwarding the http(s) traffic through your firewall for off-site monitoring should be much safer (though I would still not necessarily recommend this - simply due to best security practices...) Login is no longer transmitted over unsecured http, and people can no longer hijack your asic simply by setting a cookie to skip login.  The random 'login incorrect' messages due to file system corruption should also be a thing of the past.  Please keep in mind this will mean your password will be reset to stock default after first installing.  Also, the first boot after installation will take 2+ minutes, as the machine generates the TLS certificates.

Additionally the redesigned API has been secured w/ an access token, through which granular permissions can be assigned.  Tokens should be included with API requests in a header of the form 'Authorization: Bearer \<token\>'.

![Account Page](/docs/images/iceriver-oc_account.png)

Just as you would update the login password, PLEASE DELETE/REPLACE THIS API TOKEN if you plan on exposing your machine publicly, as it is the same across all machines by default.

<br>

### TLS certificate management
The TLS certificates (and certificate authority) for https are automatically generated on the ASIC, meaning they will cause 'Not secure' warnings in your browser since they are not from a well known authority.  While harmless, these warnings can be annoying, so the firmware provides the ability to download the CA certificate so it can be uploaded into your browsers certificate store.

![TLS Certs](/docs/images/iceriver-oc_tls.png)

To do so in Chrome, for instance, go to chrome://settings/security, click on 'Manage Certificates', select the 'Trusted Root Certification Authorities' tab (or just 'Authorities' for Linux), and click on the import button.  After restarting your browser, you should no longer see the 'Not secure' warning.

If you have multiple ASICs, you will have a different CA for each by default.  However, instead of adding each of these to your browser(s) or other devices, you can propagate a single CA across all ASICs by downloading both the CA certificate and CA key from one ASIC, uploading both files to all of your other ASICs, then regenerating the certificate on each of those other ASICs.

If you access your ASIC via a domain name or multiple IPs, you can also add these to the TLS certificate by listing them in the 'Regenerate certificate' field and clicking 'regenerate'.

<br>

### Healthcheck loop
A healthcheck loop has been added, which will automatically restart the miner or web server should either crash for any reason.

Additionally, the 'reset' executable that has been found to randomly disappear from peoples machines (even stock setups), is now packaged with the firmware, and a healthcheck loop has been added to replace/restart the file if necessary.  This should address the 30m reboot loops many people are experiencing.

<br>

# Installation
This is a standard firmware update package, including/improving on the latest IceRiver firmware, and applied just as official firmware would be.  Applying over any previous versions of this firmware should work.

However, if you run into problems, try the following process:
* Restore factory settings
* Restart the machine if it does not do so automatically
* Upload this firmware
* Once again, restart the machine if it does not do so automatically
* Force refresh the web page in your browser and/or clear the cache

Also, make sure to redo your pool settings, as they will have been reset to the default address.

<br>

# Usage Tips

### Power and metering
Laptop power supplies should generally be 19.5V with 5.5mm x 2.5mm connectors, but the amp rating can vary depending on your OC targets.  However, barrel connectors of this size tend to be rated for either 5 or 10a, and it is unlikely IceRiver used 5a options, so it would be a reasonable assumption that they used 10a (though 7.5a is another possibility).  This means that any adapter over ~220w is likely exceeding the rating of the socket, such that the plug could melt or even catch fire, if not actively cooled (even then the risk remains).  Please be extremely careful should you choose to use one of the higher power laptop charger options.

It is highly recommended you have a power meter attached to your machines, to ensure you are within your PSU limits.

<br>

### Cooling 

With more aggressive overclocks, the power stages can run very hot, so hardware modifications for improved cooling are highly recommended - including heatsinks, and better airflow.  Power stages are rated for a maximum operating temp of 125c.

Hash chips tend to perform best around 80c.

<br>

### Tuning 

CLOCK OFFSET PERCENTAGE AND HASHRATE INCREASE PERCENTAGE SHOULD BE EQUAL ON A HEALTHY MACHINE.

E.g. if your clock offset is 25% on an AL0, then your hashrate should be 500 GH/s, or 25% more than the default 400 GH/s.  If this is not the case (over an appropriate measurement window,) then it means your chips are starved for voltage.

Proper tuning is a process that takes time.  Using other peoples settings is generally not a great idea, as every machine is different.  Best practice is to start at a conservative clock offset that results in a matching hashrate increase with no voltage changes.  As you further raise your clocks in small increments (e.g. 25mhz or less), once you no longer see hashrate respond 1:1 (or maybe even start dropping), it is an indication that more voltage is needed.  

At that point, increase voltage by a single step (2mv), then see if hashrate responds.  If it does, and once again equals clock offset on a percentage basis, go back to raising clock.  Continue this back and forth between clock and voltage offsets until you reach your desired hashrate, while being mindful of temperature and power limits.

While 5m and 30m hashrates in the GUI are useful tools for directional guidance after the machine has had time to ramp up, final hashrate measurements should be done over an extended time period.  5 minute hashrate readings are quite variable, and even 30 minute hashrate readings aren't great, as you can still have a couple percent variability.  The 2hr reading in the UI should have around 1% variability from my experience, though it doesn't take hardware errors / pool rejections into account.

<br>

# Let's Talk About Hashrates
All OC firmware, including this one, only control clocks and voltages.  It is my experience that given the necessary voltage, the hashrate responds linearly on a 1:1 basis to the clock change, percentage-wise.  But in the end, all we can do is change the clock and hope the ASIC responds with the expected hashrate change.

Hashrate readings in the ASIC UI are not like those from CPU/GPU mining.  IceRiver ASICs are not counting actual hashes - they are simply estimating hashrate based on the number of shares produced * difficulty.  This is exactly how a pool measures your hashrate, but the problem is most pools decided to use way too high of a difficulty for IceRiver ASICs, which prevents reliable short term hashrate measurements - with a high diff, the share rate is low, which means wild swings in hashrate.  As a result, IceRiver released a firmware update which started using a completely different, lower internal difficulty for hashrate measurements on their own dashboard.  

Therefore, even for the same exact timeframe, you cannot reliably compare a pool hashrate measurement to the ASIC UI hashrate - they are not using the same data.  To further exacerbate this, since IceRiver machines were generating a large number of invalid shares early on, a number of pools decided to stop reporting rejected shares back to the ASIC so users would stop complaining (or switching pools), and instead report them as accepted, while still silently rejecting them on their end.  Depending on the true reject rate, this can mean a significant divergence between the ASIC hashrate and the pool hashrate, even if they were measured using the same timeframe and difficulty.

Regardless of the diff selected, hashrate measurements based on shares * difficulty are subject to swings based on luck.  The lower the share count (higher the diff), the more luck affects the hashrate, and the wilder the swings.  Thus, to have a statistically meaningful hashrate measurement, you need enough shares to reduce the effect of luck as much as possible.  The 5m reading on the ASIC are not suitable for this, especially when trying to verify the result of single digit OC changes, and short term pool readings are even worse.

You need 1200 shares just to get to an expected variance of +/- 10% with 99% confidence.  E.g. for an expected hashrate of 1TH/s, in 99/100 measurements after 1200 shares, you will have a reading between 0.9TH/s and 1.1TH/s.  You need 4800 shares to reduce that variance to +/- 5%.  Many pools are using difficulties that produce share rates in the ~5 shares/min range.  Therefore, just to get a hashrate reading with an expected variance of <= +/- 10%, you would need a 1200 / 5 = 240 minute, or 4 hour reading.  If you want a reading with an expected variance +/- 5%, you would need over 16 hours of data.  You will never be able to confirm the results of an OC level below the expected variance of a given timeframe.  For example, you cannot possibly determine whether a 5% OC is working properly in a 4 hr / 1200 share window having 10% expected variance.  Even at 16hrs / 4800 shares, the expected variance can completely cancel out a 5% OC.  

And this leads to the crux of the issue - most pools do not provide anything higher than a 24hr measurement, which at ~5 shares/minute means roughly 7200 shares, which is still a 4% expected variance.  You need 10K shares just for 3.3% variance, and about 100K shares for a 1% variance.  The 30m reading in the ASIC UI should have around a 4% variance, and the new 2hr reading should have around 1% variance, but neither reflect the pool rejects.  Therefore, the only solution then, is to find a pool that lets you set your own difficulty, so that you can generate a statistically relevant number of shares for their available timeframes.  Vipor and Herominers are examples of pools that allow this.
