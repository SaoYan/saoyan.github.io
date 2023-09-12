---
title: "Streaming Segment Parser"
date: 2019-10-14
author: yiqi
permalink: /projects/2019-10-14-streaming-segment-parser
collection: projects
tag:
- Programming
---

- [Background - ISO-BMFF Parser](#background---iso-bmff-parser)
  - [Box Parsing](#box-parsing)
  - [Segment Parsing](#segment-parsing)
- [Proposed design](#proposed-design)
  - [What's the problem?](#whats-the-problem)
  - [How to improve?](#how-to-improve)
  - [Implementation](#implementation)
- [Performance Analysis](#performance-analysis)

This was my intern project at Amazon Prime Video.

# Background - ISO-BMFF Parser

When the customer starts to play some video, the frontend client continuously requests data from the backend server. The raw data is nothing but a series of bytes that were produced through video compression and encoding. A decoder would process the raw data in order to display images and audios. ISO-BMFF parser serves as the bridge between the data and the decoder. It transforms the data in two stages: data -> box -> segment.

## Box Parsing

The first part of ISO-BMFF parser is transforming the original data into basic structures called ”box”. A box is an object containing mandatory attributes *type* and *size*. Other attributes can be included according to the specific box type. There is one special box, whose type is *mdat*. It’s a container of media data. A box can have child boxes, making a nested architecture.

```
aligned(8) Box {
    unsigned int (32) size ;
    unsigned int (32) type ;
    boxes: [] // child boxes (if existing)
    // other attributes
}
```

A typical ISO file structure looks like:  

<img src="/images/Streaming-Segment-Parser/ISO-file.jpg"  alt="iso-file"/><br>  

## Segment Parsing

The boxes described above are further parsed to become segment, either initialization segments or media segments. A video file starts with one initialization segment, with one or muliple media segments following behind. Each media segment contains a series of media samples, and each sample stores the pointer to the section of media data it contains.

<img src="/images/Streaming-Segment-Parser/segment-structures.jpg"  alt="segment-structures"/><br>  

# Proposed design

## What's the problem?

In the current implementation of the ISO-BMFF parser, the parsing process is fully synchronous. This design is insufficient in two aspects:

* Unnecessary latency is introduced because the parser has to wait for the entire segment to be downloaded before starting parsing, and the decoder can do nothing when the parser is running.
* The video decoder is capable of dealing with separate frames, so there is no need to construct a complete media segment.

## How to improve?

In order to improve the performance, the new segment parser is design to be asynchronouos. Intuitively, the parsing process "overlaps" with downloading and decoding.  

* Whenever a chunk of data is downloaded, it is sent to the parser. The parsing can starts even before the segment downloading is finished.
* Whenever some media sample is constructed within the parser, is it passed to the decoder. The decoder no longer has to wait until an entire segment is completed.

<img src="/images/Streaming-Segment-Parser/old-and-new-parser.png"  alt="old-and-new-parser"/><br>  

## Implementation

This proposed framework is implemented using event-based architecture.

<img src="/images/Streaming-Segment-Parser/event-based-impl.png"  alt="event-based-impl"/><br>  

# Performance Analysis

In order to compare the performance of the old and new parser implementation, experiments are performed on three different types of segments: audio segment, video segment, trailer segment (non-encrypted video). The experiment configuration is as follows:  

<img src="/images/Streaming-Segment-Parser/experiment-config.jpg"  alt="experiment-config"/><br>  

For each type of segment, 500 independent experiments are performed. Note that for the old parser, *tFirst* is the same as *tLast* as the entire segment is parsed as a whole.

Results of the audio segment:

<img src="/images/Streaming-Segment-Parser/performance-audio.png"  alt="performance-audio"/><br>  

Results of the video segment:

<img src="/images/Streaming-Segment-Parser/performance-video.png"  alt="performance-video"/><br>  

Results of the trailer segment:

<img src="/images/Streaming-Segment-Parser/performance-trailer.png"  alt="performance-trailer"/><br>  