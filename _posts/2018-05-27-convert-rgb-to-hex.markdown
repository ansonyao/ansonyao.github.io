---
layout: post
title:  "A tool to convert rgb to hex, conveniently"
date:   2018-05-27 10:02:44 -0700
categories: jekyll update
---

When I develop for our mobile apps, I usually get some RGB values on Zepline from our designers, which needs to be transformed into the hex code (because our mobile template developer is a hex lover!). Sounds like just trivial, right? But it does become painful when you need to do a lot of them. And I feel that the worst part is that I have to copy three times to put in the red, green, and blue values to get a result [here](https://www.rgbtohex.net/), which does not look right to me!

So I decided to build a small tool to make life easier. The nice part about the tool is that it uses regex to extract the RGB values from your input. Whether you put 0, 0, 0, or 0 0 0 or rgb(0, 0, 0) or (0, 0, 0), it will always give you the correct hex string.

I am hosting it at [https://4rgbtohex.com/](https://4rgbtohex.com/) . Feel free to add it to your tool set!

It is developed using React and material UI. Regex is used to extract values. Something like this...
{% highlight javascript %}

let regex = /^.*?\b(\d{1,3})[ ,]+(\d{1,3})[ ,]+(\d{1,3})\b.*?$/
const regexResult =regex.exec(rgb)

{% endhighlight %}

It took me quite some time to get a regex which I am happy with. The mocha based testing helped a lot when I played with the regex. I like it!  

{% highlight javascript %}
import { convertRgbToHex } from '../utils'

test('convert Rgb To hex 0, 0, 0', () => {
    expect(convertRgbToHex('0, 0, 0')).toBe('#000000');
});
  
test('convert Rgb to hex 0  0  0', () => {
    expect(convertRgbToHex('0  0  0')).toBe('#000000');
})

test('convert Rgb to hex (0  0  0)', () => {
    expect(convertRgbToHex('(0  0  0)')).toBe('#000000')
})

test('convert Rgb to hex rgb(255, 255,  64)', () => {
    expect(convertRgbToHex('rgb(255, 255,  64)')).toBe('#ffff40')
})

test('convert Rgb to hex rgba(0, 0, 0, 1)', () => {
    expect(convertRgbToHex('rgb(0,  0, 0, 1)')).toBe('#000000')
})

test('convert Rgb to hex rgba(255, 255, 255, 1)', () => {
    expect(convertRgbToHex('rgba(255, 255, 255, 1)')).toBe('#ffffff')
})

/*------------------ Error handling ----------------------*/
test('convert Rgb to hex 343 343 343', () => {
    expect(convertRgbToHex('343 343 343')).toBe('#nullnullnull')
})

test('convert Rgb to hex rgba(343 343 343)', () => {
    expect(convertRgbToHex('rgba(343 343 343)')).toBe('#nullnullnull')
})
{% endhighlight %}