---
title: Introducing tenjinreader.com
---

My thoughts on the motivation and design of [tenjinreader.com](https://tenjinreader.com)

Background
------------

There can be many different ways to learn a language.
Talking (speaking and listening) in your day to day life is the best way.

The second best is studying. Studying happens in a calm, relaxed, and distraction free environment.
It takes a lot of effort / discipline to study, but it does give better results.

I am right now focusing my efforts more on studying/reading because it is easier than talking (for me, I dont talk a lot). I learned English by reading dozens of books, and I think I can learn Japanese also.

Motivation behind the SRS
---------------------------------

I have been doing SRS reviews from the very start of Japanese learning. I find them very helpful to retain the vocabulary.

I wanted to try speech recognition for doing my reviews, so this was the main motivation behind developing a new browser based SRS app from scratch.

There were a few shortcomings in the existing tools, which led me to the current design.

### Wanikani and HouHou SRS

I used Wanikani for an year, but I realised I dont want to spend so much time learning Kanji.
Also I wanted a customised study, the flixibility to chose which words I learn first, and for which words I learn the Kanji form.

Then I moved on to HouHou SRS, it was a much better experience, as I could suspend review items which were nor relevant or difficult, and instead focus on the vocabulary which I can understand.

I used it for almost an year, it was going good but then I installed Linux as a primary OS, and now I could not longer do my reviews.

### Anki

Also during this time I started using Anki for doing the production reviews, for much of the same vocabulary as HouHou SRS.
I found it odd to use two different tools, one for recognition reviews and other one for production.

Nevertheless these are the good things about Anki

* I like the interval increment mechanism of Anki. If you answer a question correctly after 10 days, but it was due for 2 days.
Then the next review time will be calculated not by 2 days but 10 days.
I feel this is much better than the bucket system of Wanikani / HouHou SRS

* I did not have to type the answer. This was especially annoying for entering the english meanings in HouHou / Wanikani. In my case I spend a lot of time in front of computer for my job, mostly typing and I wanted to avoid putting more strain on my wrists.

  But even for hiragana input I found that many times I know the answer, and its more convenient to press the 知っている button.

**In my opinion the repetition part of the SRS should be emphasised over the game/quiz part. Its more important to have a look at the vocabulary again and again, than to answer it correctly.**

### SRS in tenjinreader

The ability to answer the reviews using voice was a really the driving factor behind the design of SRS in tenjinreader

For this feature I surveyed and tried out the existing speech to text technologies for a few months, like Kaldi and Julius.

I even got the Juluis speech recognition engine to work with a browser frontend. But the quality of recognition was unacceptable.
It would be very frustating for a user if she has to repeat the answers multiple times.
The language learning itself is a difficult process, and giving an answer should not be this difficult.

So I decided to use the Google's speech recognition (via Chrome or Chromium browser).


## Motivation behind the design of Reader

After coming to Japan I immediately realised my studies were incomplete. I had never studied the grammar and had no clue how to use the words in sentences.

I went through the [Tae Kim's awesome grammar guide](http://www.guidetojapanese.org/learn/grammar) (and still refer it). But what I really needed now was a way to not just learn useful words, but also see them used in sentences.

So soon I started with a school textbook of 5th grade containing short stories.
The content was interesting but it was quite difficult for me to read it properly, as I had to constantly refer to dictionary or online resources, and sometimes use Google translate' Camera feature just to figure out the start and end of words.

When I tried to look for computer based tools for reading Japanese, I found them to be much worse than SRS.
The reading of Japanese material is especially complicated because of the Kanji and lack of spacing between words.

I soon found japanese.io, and it was a big improvement over the earlier tools.
After using it for few months I found these problems.

* It could not handle long texts.

  I had to split my books in chapters and then split the chapters further in few pages. 
  
* The interface was not that good

  I wanted strong contrast when reading on my ebook reader (like proper black color on a white background), and dark theme when using laptop. So I had to manually tinker the page's styling.

* I wanted a way to hide the furigana for the words I already know or studying.

Overall it was manageable but not a good experience. So this was the major motivation to create this reader.

### With tenjin reader

* I can read books: 
  It even remembers my reading progress
  
* It is well integrated with SRS:
  Especially the furigana visibility

* I can change the look and feel

  The vertical text somehow feels better to read, perhaps just for aesthetic pleasure. (But why not, there are people who learn Japanese just to draw the Kanji)

* The interface of showing the word meaning is better
  
  The pop-up showing the meaning does not hide the actual word. This was a very annoying problem with japanese.io while using on ebook reader.
  
* I can mark the sentences as favourite, and review them again through "Random Fav sentence" feature.

  I can do a nested search for meaning + sentence in the same browser window.
  This is very important to not get distracted, and helps to get back where you started.
  
* It properly handles the furigana specified in the ruby《》 (especially for archaic usage in texts in public domain)

* While using an ebook reader scrolling is very difficult, it is easier to click and change the page.
