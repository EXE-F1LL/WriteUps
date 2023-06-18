>Author: @JohnHammond#6971  
>  
>I am truly sorry. I really do apologize... I _hope_ you can bear with me as I set you all loose together, on a communal collaboriative _wild goose chase_.
-------------------------------------
Within the google sheet, we see a hidden sheet if we check the bottom left:

[Check It Out YourSelf](https://docs.google.com/spreadsheets/d/17qy0Yw1_8rLOhrG5MWT8rWzpMi3_1vr3A_khcv3j6Cc/)

![Pasted image 20230615161723.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Misc/Images/Pasted%20image%2020230615161723.png)

It seems there is no way to view this as long as it is owned by somebody else, but it is possible to clone it into my own google sheet:

![Pasted image 20230615161942.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Misc/Images/Pasted%20image%2020230615161942.png)

Under our copied version, we can make the hidden sheet viewable. Within the hidden sheet seems to be a formula importing from another sheet, only viewable after turning on formulas (`View` -> `Show` -> `Formulas`).

![Pasted image 20230615162221.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Misc/Images/Pasted%20image%2020230615162221.png)
Following this page again looks like an empty sheet. Just to be sure, i exported the sheet to a csv. Without editing the saved file, I learned that the flag was present in the file name:
```
Goose Chase - Google Sheets - ... - Google Sheets - ... flag{323264294cc2a4ebb2c8d5f9e0afb0f7} - Google Sheets - Goose Chase.csv
```
It isn't obvious until I had exported, but it appears the flag was buried deep within the title name of the second sheet project. This can be verified using browser view, the flag is also present on hovering over the window:

![Pasted image 20230615162731.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Misc/Images/Pasted%20image%2020230615162731.png)

`flag{323264294cc2a4ebb2c8d5f9e0afb0f7}
