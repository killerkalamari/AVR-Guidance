# Libraries
### (and other code you expect others to use, modify, play with, and learn from)
Writing libraries that you will make available to others and expect that they will actually use presents some particular concerns that code designed for use by only yourself or your collaborators does. Even if you store it is a public github repo - a practice I wholeheartedly endorse - code that isn't meant for public consumption doesn't present the same sort of challenges. 

## Technical Matters
When you write code for your own projects, even if you try to make it generally useful (as opposed to being for a specific project, that runs on a single processor), it is natural and appropriate to limit what it will be compatible with. This might be based on your preferences on processors, the coding practices you generally use, and technical traeoffs you are willing to make (for example, I consider software serial implementations to be an abomination - for personal use, I will throw a pile of boards in the trash and redesign them if the alternative was using SoftwareSerial - but many have no problem with it). But if you're making something for use by other people - particularly if it's popular and you listen to the ones who ask for hlep, you will rapidly discover that people are using your code in different contexts and on different hardware than you had in mind when you wrote it. In the course of supporting ATTinyCore, I have been amazed and horrified by how people were using it. To maximize it's usefulness (and minimize the nubmer of people asking why it won't work in their situations), these are some guidelines:

### Use #if / #ifdef liberally
If your code would work on many parts with only small changes, use #if and #ifdef to switch implementation details depending on the hardware they are using. Ideally, if you an determine that it wont work on that hardware, you can do something like `#error "Cannot find peripheral X, which is required for this library. See the README for more information"`

### Don't assume F() will return a `__flashStringHelper`
On parts with memory mapped flash (eg, megaAVR 0-series and tinyAVR 0/1-series), it returns a pointer to a normal null terminated const char array! These platforms don't need to explicitly place constants into PROGMEM; any const is kept on flash, and can be read like a normal variable! Library code should deal with this gracefully; If C++, you should support both with different functions, and let the compiler figure out which version of the functions to build. If it's written in C, you don't have that option, so you can test for it like this:
```
#if (__AVR_ARCH__ == 103 || defined(CORE_F_MACRO_MAPPED))
//memory mapped, expect a const char*
#else
//not memory mapped, expect a const __FlashStringHelper*
#endif
```

### Test for peripherals, not chips
In the aforementioned #if's you should test for pheripherals which your code needs (by checking whether the register names are defined) not the specific parts you used or know off the top of your head will work. For example, if your library involves input capture, you might implement it on the 16-bit Timer1 peripheral on most classic AVRs have at least one of, and on a Type B timer on that the modern AVRs have, and then test for `#if defined(TCB0) || defined(TCCR1C)` - because if it had either of those things, you could be pretty confident that your library would work fine there.

The wrong way to do this is to test it on, say, the Uno, Mega, and Nano Every you have kicking around, test for those specific chips, and #error on everything else. This antipattern is incredibly common, particularly in libraries dating to the early days of Arduino. The result of this - if people like your code, at least - is that your code will get forked by someone, who will generalize your tests, quite possibly do nothing else, and then people will start using that. And you will then add new features, and users will be left to choose between your updated version, and the fork that hasn't been updated in months or years (because the guy who forked and changed a few lines long ago lost interest), and your up-to-date version with your spiffy new features, which only supports a couple of parts. Don't be one of these authors - help your users and capture more of the prestige. 

## Non-technical Matters
There are also some issues that have nothing to do with the code itself that you need to consider. 

### This isn't your job (well, unless it is) 
If the code is not in support of a product you're selling, you are under no obligation to help people with it. Whem someone asks you to add some feature that you know will take a lot of work, and that you don't think are useful useful, don't hesitate to close it and say that this feature does not justify the time it would take, but that you'd welcome a pull request if they want to write it (assuming you actually would, at least). Sometimes you can get people to write your code for you! 

Of course, if you're selling something that relies on that code, there's an expectation that you'll provide a whole other level of support: Those people complaining are not random internet people whining about the free code not doing everything they want - they're your *customers.*

### Your code reflects on you
People will judge your personality and ability based on what they see in your code. Potential employers will see it when they search for you on social media, etc. So, don't be offensive or political in the comments (if nothing else, this ensures that you won't get people contacting you just to complain about the political comment you made because you were listening to some political show or podcast in the backhround when you wrote it). The norms even within one country can vary more than many people realize - and the internet is global. That said, nobody is going to be particularly bothered by occasional profanity in a comment - particularly where it seems warranted. 

Particularly if you are in the software field, or place particular value on other people thinking you are competent, you also need to think about how the code reflects on your ability and programming habits. Inconsistent indentations, misspelled words in variable names amd things like that make you look sloppy, particularly in code you've made a point of documenting and making available for others. Doing things in a clumsy way - something that reveals that you don't know the Right Way to do something - that reflects on your knowledge of the language. If you are doing it in the more verbose and and "newbie" way for a reason, a comment to that effect can preempt this (and the helpful user's pull request that replaces it with the more "graceful" way).

All these considerations also apply to repos that are visible, but not optimized for use by others - though of course not to the same extent. If you have a mix of "released to public for re-use" repos and "something I did for myself, but might as well let people see it", it's a good idea to mention at the top of the README for those in the latter category something to the effect of "The code in this repo was made for personal projects, and is not expected to be useful to others".
