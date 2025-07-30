# bypassantidebug
TL;DR: You are going to get fucked by sites detecting your devtools. You need to know what techniques are used to bypass them.

Many sites use some sort of debugger detection to prevent you from looking at the important requests made by the browser.

You can test the devtools detector here: https://blog.aepkill.com/demos/devtools-detector/ (does not feature source mapping detection) Code for the detector found here: https://github.com/AEPKILL/devtools-detector

How are they detecting the tools?
One or more of the following methods are used to prevent devtools in the majority of cases (if not all):

1. Calling debugger in an endless loop. This is very easy to bypass. You can either right click the offending line (in chrome) and disable all debugger calls from that line or you can disable the whole debugger.

2. Attaching a custom .toString() function to an expression and printing it with console.log(). When devtools are open (even while not in console) all console.log() calls will be resloved and the custom .toString() function will be called. Functions can also be triggered by how dates, regex and functions are formatted in the console.

This lets the site know the millisecond you bring up devtools. Doing const console = null and other js hacks have not worked for me (the console function gets cached by the detector).

If you can find the offending js responsible for the detection you can bypass it by redifining the function in violentmonkey, but I recommend against it since it's often hidden and obfuscated. The best way to bypass this issue is to re-compile firefox or chrome with a switch to disable the console.

3. Invoking the debugger as a constructor? Looks something like this in the wild:

function _0x39426c(e) {
    function t(e) {
        if ("string" == typeof e)
            return function(e) {}
            .constructor("while (true) {}").apply("counter");
        1 !== ("" + e / e).length || e % 20 == 0 ? function() {
            return !0;
        }
        .constructor("debugger").call("action") : function() {
            return !1;
        }
        .constructor("debugger").apply("stateObject"),
        t(++e);
    }
    try {
        if (e)
            return t;
        t(0);
    } catch (e) {}
}
setInterval(function() {
    _0x39426c();
}, 4e3);
This function can be tracked down to this script

This instantly freezes the webpage in firefox and makes it very unresponsive in chrome and does not rely on console.log(). You could bypass this by doing const _0x39426c = null in violentmonkey, but this bypass is not doable with heavily obfuscated js.

Cutting out all the unnessecary stuff the remaining function is the following:

setInterval(() => {
    for (let i = 0; i < 100_00; i++) {
        _ = function() {}.constructor("debugger").call(); // also works with apply
    }
}, 1e2);
Basically running .constructor("debugger").call(); as much as possible without using while(true) (that locks up everything regardless). This is very likely a bug in the browser.

4. Detecting window size. As you open developer tools your window size will change in a way that can be detected. This is both impossible to truly circumvent and simultaneously easily sidestepped. To bypass this what you need to do is open the devtools and click settings in top right corner and then select separate window. If the devtools are in a separate window they cannot be detected by this technique.

5. Using source maps to detect the devtools making requests when opened. See https://weizmangal.com/page-js-anti-debug-1/ for further details.

How to bypass the detection?
I have contributed patches to Librewolf to bypass some detection techniques. Use Librewolf or compile Firefox yourself with my patches.

Get librewolf at https://librewolf.net/
Go to about:config
Set librewolf.console.logging_disabled to true to disable method 2
Set librewolf.debugger.force_detach to true to disable method 1 and method 3
Make devtools open in a separate window to disable method 4
Disable source maps in developer tools settings to disable method 5
Now you have completely undetectable devtools!
