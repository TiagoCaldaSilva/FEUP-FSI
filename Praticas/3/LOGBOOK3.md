# Trabalho realizado na Semana #3

## Identificação

- Mozilla is aware of a critical vulnerability affecting Firefox, Thunderbird & SeaMonkey users. Affected is the function document.write of the component DOM. 
- When JavaScript is enabled, allows remote attackers to execute arbitrary code. The manipulation with an unknown input leads to a memory corruption vulnerability. 
- The software performs operations on a memory buffer, but it can read from or write to a memory location that is outside of the intended boundary of the buffer.
- According to the reports, this flaw can potentially allow an attacker to exploit the user's machine through the browser by making it run arbitrary code without user interaction - a classic drive-by vulnerability.
- CWE is classifying the issue as CWE-119: Improper Restriction of Operations within the Bounds of a Memory Buffer.


## Catalogação

- The trojan was initially reported by Morten Kråkvik with Telenor SOC.
- It is classified as a critical vulnerability, having a CVSS Score of 9.3
- After 2 days, an exploit had already been disclosed. The initial estimated underground price was around $25k-$100k.
- The best possible mitigation is suggested to be upgrading to the latest version. A possible mitigation was published 1 day after the disclosure of the vulnerability. 
- Was fixed in, Firefox 3.5.15, Firefox 3.6.12, SeaMonkey 2.0.10, Thunderbird 3.0.10, Thunderbird 3.1.6


## Exploit

- The exploitation doesn't require any form of authentication. Successful exploitation requires user interaction by the victim. Technical details and a public exploit are known.
- There are 4 exploits available, being 2 of them Remote exploits and the remaining 2 DoS exploits. The Remote exploits used the Windows platform, the other ones used multiple platforms.
- Interleaving document.write and appendChild can lead to duplicate text frames and overrunning of text run buffers.
- This exploit is written in JavaScript. When a user visits a web page containing the exploit, it could download other malware.

Exploit example:
```html
<html><body>
<script>

  function G(str){
    var cobj=document.createElement(str);
    document.body.appendChild(cobj);
    cobj.scrollWidth;
  }

  function crashme() {
    document.write("fooFOO");
    G("a");
    document.write("<a lang></a>a");
    G("base");
    document.write("barBAR");
    G("audio");
  }
</script>
<script>crashme();</script>
</body>
</html>
```

## Ataques

- This vulnerability affects various users hence the urge to fix it rapidly. There is a complete loss of system protection resulting in the entire system being compromised.
- Users who visited an infected site could have been affected by the malware through the vulnerability. 
- The Nobel Peace Prize website was serving on October 25, 2010 a zero-day exploit against Firefox users.  
- When people accessed the Nobel Peace Prize site they were diverted onto an attack server located in Taiwan which delivered a JavaScript exploit.
