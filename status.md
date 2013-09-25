## Emoncms.org status

#### Troubleshooting

**Incorrect password** - returned when your sure your using the same password as pre March 16th emoncms upgrade. Try clearing your browser cache. The new login form has an update to the way the password field is sent which allows characters such as & (the url is encoded using urlencode).

**Username does not exist** - Try entering your username without @ and . character underscore is allowed.

**Blank accounts page** - clear cache again.

Any further login queries please contact trystanlea@openenergymonitor.org

---

#### Browser Compatibility

**Chrome Ubuntu 23.0.1271.97** - developed with, works great.

**Chrome Windows 25.0.1364.172** - quick check revealed no browser specific bugs.

**Firefox Ubuntu 15.0.1** - no critical browser specific bugs, but movement in the dashboard editor is much less smooth than chrome. 

**Internet explorer 9** - works well with compatibility mode turned off. F12 Development tools -> browser mode: IE9. Some widgets such as the hot water cylinder do load later than the dial.

**IE 8, 7** - not recommended, widgets and dashboard editor do not work due to no html5 canvas fix implemented but visualisations do work as these have a fix applied.

---

#### September 2013

Emoncms.org is on the latest emoncms master branch with timestore enabled. 

#### 16th March 2013

Emoncms.org has just been upgraded to latest development branch of emoncms https://github.com/emoncms/emoncms/tree/dev, there are known issues with dashboard (fixed?) and multigraph which are being worked on, notice any other bugs please email me: trystanlea@openenergymonitor.org. If the accounts page is empty, try refreshing your browser cache.

**Input name change:** Input names where changed from node10_1 convention to 1 with the nodeid in the nodeid field.

#### 17th March 2013

**Username change:** All . characters have been removed from usernames as the . character conflicts with the new routing implementation where emoncms thinks that the part after the . is the format the page should be in.

**Multigraph fixed:** multigraphs should now work as normal.

Graph of input processing performance improvement as part of the latest emoncms development branch: [http://t.co/zrMeBPo2cT](http://t.co/zrMeBPo2cT)

#### 21st March 2013

**input/post CSV error** all csv input data was parsed as positive values due to - character being stripped out. This has now been fixed, feeds should now report the correct value. 

#### 22 March 2013

Merged changes from github dev branch
