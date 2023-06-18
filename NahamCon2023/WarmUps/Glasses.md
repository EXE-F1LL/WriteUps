>Author: @JohnHammond#6971  
>  
>Everything is blurry, I think I need glasses!
---------------------------------
Visiting the page gives us a very non-interactive page where we might buy glasses.
Checking the source code with ctrl+U is an absolute mess. However, since everything I interact with does not take me anywhere new, I want to view html code on the page. Using right click to inspect has an interesting interaction:

![Pasted image 20230615234612.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Warmup/Images/Pasted%20image%2020230615234612.png)

What's even more interesting is this does nothing. I still get to inspect anyways.

After leafing through the code a little bit, I do indeed find a piece that is interesting:
```html
<p class="lead">
	Can't see things because they are too blurry? Is your vision foggy, and everything looks like
	<em>this?</em>
	<span style="color: transparent;text-shadow: 0 0 5px rgba(0,0,0,0.09);text-decoration: line-through;">flag½₧8084e4530cf649814456f2a291eb81e9½―</span>
	<br>
	Test your vision with glasses! Try on this pair and see if you can see the previous sample text... you might find a lot of value! Imagine the world where you can see clearly!
```
Here is the same selection displayed on the webpage:

![Pasted image 20230615234853.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Warmup/Images/Pasted%20image%2020230615234853.png)

The color of our flag is transparent on top of white, and perhaps might be microscopically tiny as well. Either way, it is evident once we inspect the properties on the page.
`flag{8084e4530cf649814456f2a291eb81e9}`

 - Author : SpacerJa
