## Further testing of Cline

**TLDR; Cline wasn't great at optimising my app's CSS and got stuck launching browser to review changes.**

I continued to test operation of cline using the gemini model, this time to try and optimise the (LNRS Toolkit)[https://opendata.westofengland-ca.gov.uk/pages/lnrs-application/?headless=true] which I built. This is an app built on (opendatasoft's platform)[https://opendata.westofengland-ca.gov.uk/pages/homepage/] using ODS's widgets and AngularJS. I wanted to see if cline could help to optimise the operation and appearance of the app.

I gave the model my CSS and HTML files and asked it to review the code. It made sevaral changes, as I just auto - approved the changes and modified the files. I copied the files into the ODS backoffice and then published the app. Unfortunately many of the changes were wrong, and the elements appeared in the wrong places.

I asked cline to review the web page but it got stuck on launching the browser. After several loops of trying to resolve, it gave up. And I had burned through all my Gemini 2.5 Flash tokens so have to switch to Gemini 2.0. Not sure if Claude would have been better - but expensive in any case.

However, It is quite amazing to see this tool in action and how potentially powerful it is. You can see it writing code, thinking and reasoning, and it shows your API token usage and cost in real time. There seems no doubt that it will continue to improve, so definitely worth continuing to learn. I like that it is integrated into VS Code.