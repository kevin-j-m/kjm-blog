+++
title = "Enough Coverage To Beat The Band"
date = 2020-10-19T08:12:10-04:00
tags = ["ruby", "conference", "rubyconf"]
categories = []
summary = "Using Coverage To Stage A Concert Tour"
aliases = ["/coverage"]
+++

## Presentation Resources

Kevin's slides are available for review on SpeakerDeck. The code examples that accompanied the talk are available on [Github](https://github.com/kevin-j-m/ruby_cover_band).

## Code Coverage Modes

Each mode answers a different question about the code that was run under coverage:

* Lines - how many times was each line executed?
* Oneshot Lines - which lines were executed?
* Methods - how many times was each method executed?
* Branches - how many times was each conditional executed?

### Lines
{{< rawhtml >}}
<p class="float-image coverage-emoji">
🎸
</p>
{{< /rawhtml >}}

This is the “classic” implementation of providing coverage. Each relevant line, that is those that aren’t things like empty lines or “end” statements, has a counter that is incremented each time the line is visited in code execution while coverage is running. At the conclusion, you will see how many times each line is executed.

#### Benefits

* This is the default mode for coverage.
* Most of the time, this option will provide you with the information you want.

### Oneshot Lines

{{< rawhtml >}}
<p class="float-image coverage-emoji">
🎹
</p>
{{< /rawhtml >}}

Similar to Line Coverage, this also documents that a relevant line was executed while coverage was running. However, it’s a binary report of whether it was executed or not. It will not tell you how often. This may be sufficient in many cases, and comes with the benefit of being more performant every subsequent time a particular line of code is executed under coverage.

#### Benefits

* Oneshot provides you with nothing more than if a line of application code is executed in a test suite.
* As long as being constrained to knowing if something ran or not, and not knowing how often, is sufficient, Oneshot Line Coverage provides the same feedback as Line Coverage with better performance.

### Methods

{{< rawhtml >}}
<p class="float-image coverage-emoji">
💡
</p>
{{< /rawhtml >}}

Method Coverage brings the granularity of Line Coverage up to a coarser grain. Rather than tracking individual lines, it’s concerned with whether a particular method is executed. It can be a 10 line method where the first line is the only line ever executed. Method Coverage will still consider that as executed the same as a 20 line method where each line is executed.

#### Benefits

* This has a targeted focus to be able to answer a more specific question - is this method executed? - with easier to process feedback. 

### Branches

{{< rawhtml >}}
<p class="float-image coverage-emoji">
🎤
</p>
{{< /rawhtml >}}

Branch Coverage tracks execution of different conditional paths and documents how often those different paths are run. The unique benefit that this provides over Line Coverage is in conditionals that execute multiple code paths in a single line, such as ternary statements. You may have a part of that conditional that’s never run or tested, but you wouldn’t know that if you’re relying on Line Coverage alone.

#### Benefits

* It provides a different frame of reference than Line Coverage, which ends up being either coarser or more granular than line coverage in different situations.
* For conditionals that lay out multiple code paths on a single line, this provides feedback on their individual execution where Line Coverage only considers whether any part of the line was run.
* When interested in conditionals, and only conditionals, it has less noise than Lines Coverage.

## The Gnar Company

If you’d like to discuss how [The Gnar Company](https://www.thegnar.co/about.html) can work with you on your technical challenges, [let us know](https://www.thegnar.co/hire-us.html).

## Presentation Photo Credits

All photos used in the presentation are from the band [Nine Inch Nails](https://www.nin.com/), and released on
their [flickr](https://www.flickr.com/photos/nineinchnails/) account with a [Creative Commons Attribution-NonCommercial-ShareAlike 2.0 Generic](https://creativecommons.org/licenses/by-nc-sa/2.0/) license.

Below I've embedded all the images used to link directly to their original
source.

### Introduction
{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2738980841/" title="Nine Inch Nails Live @ Lollapalooza - Chicago, IL, 8.03.08"><img src="https://live.staticflickr.com/2222/2738980841_dbc739b957_c.jpg" width="800" height="533" alt="Nine Inch Nails Live @ Lollapalooza - Chicago, IL, 8.03.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3639150601/" title="Nine Inch Nails live @ PNC Bank Arts Center, Holmdel, NJ, 6.06.09"><img src="https://live.staticflickr.com/3606/3639150601_409c17e154_c.jpg" width="800" height="534" alt="Nine Inch Nails live @ PNC Bank Arts Center, Holmdel, NJ, 6.06.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

### Lines Coverage

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3573650628/" title="Nine Inch Nails Live @ Shoreline Amphitheatre - Mountain View, CA, 5.22.09"><img src="https://live.staticflickr.com/2455/3573650628_75ed822d85_c.jpg" width="800" height="533" alt="Nine Inch Nails Live @ Shoreline Amphitheatre - Mountain View, CA, 5.22.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3573650014/" title="Nine Inch Nails Live @ Shoreline Amphitheatre - Mountain View, CA, 5.22.09"><img src="https://live.staticflickr.com/3340/3573650014_cf942da814_c.jpg" width="500" height="750" alt="Nine Inch Nails Live @ Shoreline Amphitheatre - Mountain View, CA, 5.22.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/10722222236" title="Nine Inch Nails Live: Tension 2013"><img src="https://live.staticflickr.com/2841/10722222236_c5e6dd1150_c.jpg" width="533" height="800" alt="Nine Inch Nails Live: Tension 2013"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3639160537/" title="Nine Inch Nails live @ Bonnaroo Festival in Manchester, TN, 6.13.09"><img src="https://live.staticflickr.com/3331/3639160537_b3f8ea5158_c.jpg" width="533" height="800" alt="Nine Inch Nails live @ Bonnaroo Festival in Manchester, TN, 6.13.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3572799825/" title="Nine Inch Nails Live @ Cricket Wireless Amphitheater - Chula Vista, CA, 5.16.09"><img src="https://live.staticflickr.com/2432/3572799825_c091e87ff3_c.jpg" width="800" height="533" alt="Nine Inch Nails Live @ Cricket Wireless Amphitheater - Chula Vista, CA, 5.16.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3639938714/" title="Nine Inch Nails live @ Starlight Theatre, Kansas City, MO, 5.27.09"><img src="https://live.staticflickr.com/3664/3639938714_81c6079524_c.jpg" width="800" height="534" alt="Nine Inch Nails live @ Starlight Theatre, Kansas City, MO, 5.27.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

### Oneshot Lines Coverage

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3639113025/" title="Nine Inch Nails Live @ Sasquatch Festival in George, WA, on 5.24.09"><img src="https://live.staticflickr.com/2460/3639113025_dee5235b96_c.jpg" width="800" height="534" alt="Nine Inch Nails Live @ Sasquatch Festival in George, WA, on 5.24.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3032772287/" title="Nine Inch Nails Live @ Bell Centre - Montreal, QC, 11.12.08"><img src="https://live.staticflickr.com/3057/3032772287_517c1d7b41_c.jpg" width="800" height="533" alt="Nine Inch Nails Live @ Bell Centre - Montreal, QC, 11.12.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2739834768" title="Nine Inch Nails Live @ Air Canada Centre - Toronto, ON, 8.05.08"><img src="https://live.staticflickr.com/3252/2739834768_7d40ed685e_c.jpg" width="800" height="533" alt="Nine Inch Nails Live @ Air Canada Centre - Toronto, ON, 8.05.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3573642602" title="Nine Inch Nails Live @ Santa Barbara Bowl - Santa Barbara, CA, 5.21.09"><img src="https://live.staticflickr.com/3606/3573642602_c343bd27b4_c.jpg" width="800" height="533" alt="Nine Inch Nails Live @ Santa Barbara Bowl - Santa Barbara, CA, 5.21.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3212245282" title="Nine Inch Nails Live @ Razzmatazz - Barcelona, Spain, 2.19.07"><img src="https://live.staticflickr.com/3463/3212245282_e1fee55ee5_z.jpg" width="640" height="480" alt="Nine Inch Nails Live @ Razzmatazz - Barcelona, Spain, 2.19.07"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3573605580/" title="Nine Inch Nails Live @ Cricket Wireless Amphitheater - Chula Vista, CA, 5.16.09"><img src="https://live.staticflickr.com/3601/3573605580_2dafcc463c_z.jpg" width="640" height="427" alt="Nine Inch Nails Live @ Cricket Wireless Amphitheater - Chula Vista, CA, 5.16.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3211397961/" title="Nine Inch Nails Live @ Olympia - Paris, France, 2.21.07"><img src="https://live.staticflickr.com/3481/3211397961_2c19e04509_z.jpg" width="640" height="480" alt="Nine Inch Nails Live @ Olympia - Paris, France, 2.21.07"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2827641506/" title="Nine Inch Nails Live @ Seagate Convention Center - Toledo, OH, 8.25.08"><img src="https://live.staticflickr.com/3148/2827641506_d102315760_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ Seagate Convention Center - Toledo, OH, 8.25.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2768600347/" title="Nine Inch Nails Live @ Gwinnett Arena - Duluth, GA, 8.13.08"><img src="https://live.staticflickr.com/3147/2768600347_8a7059bf75_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ Gwinnett Arena - Duluth, GA, 8.13.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2997174649/" title="Nine Inch Nails Live @ Arena Santiago - Santiago, Chile, 10.04.08"><img src="https://live.staticflickr.com/3276/2997174649_c8067aa285_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ Arena Santiago - Santiago, Chile, 10.04.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

### Methods Coverage

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/10722216364" title="Nine Inch Nails Live: Tension 2013"><img src="https://live.staticflickr.com/3723/10722216364_e307c51d3b_z.jpg" width="640" height="427" alt="Nine Inch Nails Live: Tension 2013"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3572822083/" title="Nine Inch Nails Live @ The Pearl - Las Vegas, NV, 5.18.09"><img src="https://live.staticflickr.com/3298/3572822083_88b2ccae93_z.jpg" width="640" height="427" alt="Nine Inch Nails Live @ The Pearl - Las Vegas, NV, 5.18.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/10722275244/" title="Nine Inch Nails Live: Tension 2013"><img src="https://live.staticflickr.com/3694/10722275244_34e8865f69_z.jpg" width="640" height="427" alt="Nine Inch Nails Live: Tension 2013"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/10722195315/" title="Nine Inch Nails Live: Tension 2013"><img src="https://live.staticflickr.com/7459/10722195315_2e3a458395_z.jpg" width="640" height="427" alt="Nine Inch Nails Live: Tension 2013"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/10722461283/" title="Nine Inch Nails Live: Tension 2013"><img src="https://live.staticflickr.com/5539/10722461283_68242773f4_z.jpg" width="640" height="424" alt="Nine Inch Nails Live: Tension 2013"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/10722293856/" title="Nine Inch Nails Live: Tension 2013"><img src="https://live.staticflickr.com/7387/10722293856_eedc277cda_z.jpg" width="427" height="640" alt="Nine Inch Nails Live: Tension 2013"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2738931283/" title="Nine Inch Nails Live @ Pemberton Festival - Pemberton, BC, 7.25.08"><img src="https://live.staticflickr.com/2353/2738931283_2bcaac316c_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ Pemberton Festival - Pemberton, BC, 7.25.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2739771096/" title="Nine Inch Nails Live @ Pemberton Festival - Pemberton, BC, 7.25.08"><img src="https://live.staticflickr.com/2103/2739771096_6fda018740_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ Pemberton Festival - Pemberton, BC, 7.25.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2826802685/" title="Nine Inch Nails Live @ Q Arena - Cleveland, OH, 8.22.08"><img src="https://live.staticflickr.com/3100/2826802685_f425c65f80_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ Q Arena - Cleveland, OH, 8.22.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3202016537/" title="Nine Inch Nails Live @ Coliseum - Lisbon, Portugal, 2.11.07"><img src="https://live.staticflickr.com/3079/3202016537_6c2dd2e850_z.jpg" width="480" height="640" alt="Nine Inch Nails Live @ Coliseum - Lisbon, Portugal, 2.11.07"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2738948949/" title="Nine Inch Nails Live @ Key Arena - Seattle, WA, 7.26.08"><img src="https://live.staticflickr.com/3223/2738948949_a6e508137f_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ Key Arena - Seattle, WA, 7.26.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

### Branches Coverage

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3032765125/" title="Nine Inch Nails Live @ DCU Center - Worcester, MA, 11.09.08 (with surprise guest Peter Murphy)"><img src="https://live.staticflickr.com/3044/3032765125_0cea90d73b_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ DCU Center - Worcester, MA, 11.09.08 (with surprise guest Peter Murphy)"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2997970466/" title="Soundcheck in Buenos Aires, Argentina - 10.01.08"><img src="https://live.staticflickr.com/3143/2997970466_c27cdaef49_z.jpg" width="427" height="640" alt="Soundcheck in Buenos Aires, Argentina - 10.01.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2997970974/" title="Soundcheck in Buenos Aires, Argentina - 10.01.08"><img src="https://live.staticflickr.com/3012/2997970974_2cb4dec9ba_z.jpg" width="427" height="640" alt="Soundcheck in Buenos Aires, Argentina - 10.01.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2739752862/" title="Nine Inch Nails Soundcheck - Pemberton, BC, 7.24.08"><img src="https://live.staticflickr.com/2307/2739752862_346b683bcb_z.jpg" width="640" height="426" alt="Nine Inch Nails Soundcheck - Pemberton, BC, 7.24.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2738913701/" title="Nine Inch Nails Soundcheck - Pemberton, BC, 7.24.08"><img src="https://live.staticflickr.com/3201/2738913701_9b991d6e95_z.jpg" width="640" height="426" alt="Nine Inch Nails Soundcheck - Pemberton, BC, 7.24.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/2997202615/" title="Soundcheck in Mexico City, Mexico - 10.17.08"><img src="https://live.staticflickr.com/3061/2997202615_04151a1234_z.jpg" width="640" height="426" alt="Soundcheck in Mexico City, Mexico - 10.17.08"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3017100564/" title="Nine Inch Nails Live @ House Of Blues - Atlantic City, New Jersey, 11.06.08 (with surprise guest Peter Murphy)"><img src="https://live.staticflickr.com/3017/3017100564_0a48c0b815_z.jpg" width="640" height="426" alt="Nine Inch Nails Live @ House Of Blues - Atlantic City, New Jersey, 11.06.08 (with surprise guest Peter Murphy)"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

### Closing

{{< rawhtml >}}
<a data-flickr-embed="true" data-header="true" data-footer="true" href="https://www.flickr.com/photos/nineinchnails/3639141695/" title="Nine Inch Nails live @ DTE Energy Music Theatre, Clarkston, MI, 5.31.09"><img src="https://live.staticflickr.com/3360/3639141695_836a8f0745_z.jpg" width="427" height="640" alt="Nine Inch Nails live @ DTE Energy Music Theatre, Clarkston, MI, 5.31.09"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}
