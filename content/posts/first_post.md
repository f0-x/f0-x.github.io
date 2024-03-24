---
title: "My First Post"
date: 2022-02-04T17:12:57+05:45
draft: false
tags : ["oxHugo"]
categories : ["hugo"]
---

Dummy First Post


This is my first post body. I am excited.😎 😍

```C { linenos=table, linenostart=7, hl_lines=["1","3-5"] }
#include <stdio.h>         //include info about standard library

int main()                    // define a function named main that receives no
		      // argument values.
{
    printf("hello, world from Ox-Hugo\n");   //main calls library function printf
}                             // to print the sequence of characters;
			      // \n represents a newline character
```

{{< figure src="/images/avatar.png" >}}

```EmacsLisp
(message "Hello World Cup")
```


## Latex Section {#latex-section}

\\[ c = \pm\sqrt{a^2 + b^2} \\]

\\[
f(x)=\int\_{-\infty}^{\infty} \hat{f}(\xi) e^{2 \pi i \xi x} d \xi
\\]

If \\(a^2=b\\) and \\( b=2 \\), then the solution must be either
\\[ a=+\sqrt{2} \\] or \\[ a=-\sqrt{2} \\]

\\(\left(\LARGE{AB}\right)\\)


### LaTeX Environments {#latex-environments}

\\[\begin{matrix}
   a & b \\\\
   c & d
\end{matrix}\\]

\\[\begin{equation}
C = W\log\_{2} (1+\mathrm{SNR})
\end{equation}\\]

{{< admonition type="danger" >}}
This is a warning.
{{< /admonition >}}


{{< gist spf13 7896402 >}}

{{< mermaid >}}
graph LR;
    A[Hard edge] -->|Link text| B(Round edge)
    B --> C{Decision}
    C -->|One| D[Result one]
    C -->|Two| E[Result two]
{{< /mermaid >}}

{{< mermaid >}}
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
{{< /mermaid >}}

{{< mermaid >}}
gantt
    dateFormat  YYYY-MM-DD
    title Adding GANTT diagram functionality to mermaid
    section A section
    Completed task            :done,    des1, 2014-01-06,2014-01-08
    Active task               :active,  des2, 2014-01-09, 3d
    Future task               :         des3, after des2, 5d
    Future task2               :         des4, after des3, 5d
    section Critical tasks
    Completed task in the critical line :crit, done, 2014-01-06,24h
    Implement parser and jison          :crit, done, after des1, 2d
    Create tests for parser             :crit, active, 3d
    Future task in critical line        :crit, 5d
    Create tests for renderer           :2d
    Add to mermaid                      :1d
{{< /mermaid >}}

{{< typeit >}}
This is a *paragraph* with **typing animation** based on [TypeIt](https://typeitjs.com/)...
{{< /typeit >}}


