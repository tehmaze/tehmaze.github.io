---
layout: post
title: "Digital Mobile Radio"
categories: [blog,radio]
share: true
comments: true
excerpt: What do you get if you combine modern day technologies with motivated HAMs?
tags: [radio,dmr,coding]
author: maze
image:
  feature: /posts/radio.jpg
date: 2016-03-23T10:07:00+01:00
---
Lately I've become more involved in the HAM radio DMR community. Digital
Mobile Radio, or DMR, is a Time Division Multiple Access (TDMA) digital voice
system suitable for VHF and up. Most HAM radio applications utilise the UHF
bands with typically a frequency split of 7.6 MHz, very much like traditional
FM repeaters. More and more DMR enabled repeaters are popping up and there have
been competing networks to link the repeaters up over the internet.

## Other digital voice standards

There are many other digital voice standards operated by the HAM radio
community such as TETRA, D-STAR, C4FM and many more. DMR is a relatively new
technology, developed as an open standard by the European Telecommunications
Standards Institute (ETSI). Comparing the different technologies is beyond
the scope of this blog post, but DMR is a relatively new technology built
for commercial use. This means there is still plenty of room for
experimentation to make it suitable for HAM radio use whereas there are not
many new developments on the D-STAR network.

## DMR Repeaters

DMR repeaters are mostly built on using ready-existing, commercial repeaters
form various vendors like [Motorola][], [Hytera][], [RadioActivity][], etc.
But the HAM community has started to experiment with more open alternatives,
like the excellent [Multi-Mode Digital Voice Modem][] (MMDVM), [DVMEGA][]
and [DV4mini][] projects. The MMDVM project aims to build a full capable,
open source DMR repeater system while the DVMEGA and DV4mini allow end users
to run their own local DMR hot spot.

## Linked repeaters

<div class="post-image-large">
	<a href="http://www.cqdmrmap.com/"><img src="/images/cqdmrmap.png" alt="CQ DMR map"></a>
	<label>CQ DMR map &copy; EA2CQ</label>
</div>

Various projects exist today that link up the various DMR repeater systems.
This can be somewhat confusing to the end user, because not all systems link up
to the other existing systems, so there are multiple views of what is the
global DMR platform. You can get an overview of how the various networks are
spread on the [http://www.cqdmrmap.com/](http://www.cqdmrmap.com/) site.

More and more sites are adapting the Brandmeister DMR technology. Brandmeister
is driven by a very talented group of HAM radio operators from all over the
world. What makes Brandmeister so powerful? The people driving the project.

In my humble opinion Brandmeister is a prime example of what is possible
when you combine modern day technologies with a group of driven HAM radio
operators. Brandmeister offers support for a [very wide variety of
vendors](http://www.dstar.su/tools/panel/list.htm) and is fully decentralized.
Also, Brandmeister is promoting experimentation with home built solutions by
offering [Homebrew protocol](https://bm.pd0zry.nl/index.php/Protocols)
support, so HAM radio operators can plug in to the Brandmeister network.

## Voice synthesis

DMR uses digital speech synthesis using the Advanced Multi-Band Excitation, or
AMBE codec developed by [Digital Voice Systems, Inc.](http://www.dvsinc.com).
Initially the AMBE codec is what scared me away from DMR, because I think it's
very important that we keep the HAM radio world free from patented technologies
that carry crazy license fees. Now over the last couple of months, DVSI has
made the AMBE chips more affordable and Chinese radio manufacturing companies
like [Tytera][] are implementing low-cost software versions of the AMBE codec
in their radio systems.

Luckily, the DMR specification leaves plenty of room for other voice codecs to
be carried over the DMR network. So hopefully, the HAM radio will start
experimenting with open voice standards like [Codec2][] or [Opus][].

## Adressing on the DMR network

Addressing on the DMR network uses 24 bit identifiers to uniquely identify
actors on the network. There are two standard modes that carry voice
information (indicated by the Full Link Control Opcode or FLCO) that indicate
wether a voice transmission is destined for a group, or for a (private) user
to user transmission. If a group voice transmission is sent, the destination
identifier is usually referred to as a *Talk Group*.

## Brandmeister

Since HAM radio operators don't want to get vendor locked, a team from Russia
has started the [Brandmeister][] project to link up repeaters using different
technologies. More talented developers from all over the world have joined the
Brandmeister project and the development team continues to expand.

<div class="post-image-large">
	<a href="http://www.brandmeister.network/"><img src="/images/bm-growth.png" title="Brandmeister network growth © PD0ZRY"></a>
	<label>Brandmeister network growth, © PD0ZRY</label>
</div>

Starting off small, the Brandmeister project now has over 250 repeaters linked
up from all over the world. This is still a relatively small network compared
to D-STAR which has over 1000 repeaters linked, but the network growth rate is
fenominal. The repeaters setup a link to a regional master, that
has a full mesh with all the other regional masters around the world. This
allows anyone on the Brandmeister DMR network to open up remote *Talk Groups* 
and talk with the other side of the world as if they are on the same local
repeater.

## Hose line

Piggy backing on the full mesh routing capabilities, I have started to develop
a web based front end dubbed [Hose line][] which allows users to listen in to
the voice streams that are on the network. The engine is entirely built in
[Go][] and Javascript. This is how it all works.

<p>
  <div class="post-image-large">
	<a href="http://hose.brandmeister.network/"><img src="/images/hoseline.png" title="Hose line"></a>
	<label>Hose line &copy; PD0MZ</label>
  </div>
</p>

### Full mesh routing

All the regional masters on the Brandmeister network have a list of what other
regional masters exist on the network. If a user on the Brandmeister DMR network
starts a voice conversation to a *Talk Group*, all of the voice transmissions are
relayed to all the other regional masters on the network. In turn, the regional
master relays all of these packets to all of the repeaters. The repeaters keep
track of what *Talk Groups* are local to them, or what *Talk Groups* are recently
activated and decides if the voice frame will be relayed to the radio interface
or not.

### Streaming

The [Hose line][] project uses the full mesh routing capabilities of the
Brandmeister DMR network to receive voice frames from all the regional masters
around the world. When a voice frame is received for a *Talk Group*, Hose line
check if there are subscribers listening to the stream and decodes the voice
frames *on-demand* so it can relay it as plain audio to the user's browser. All
of this is powered by a very simple publish-subscribe (or "pubsub") bus written
in the [Go][] programming language.

[Motorola]: http://www.motorolasolutions.com/en_us/products/mototrbo-systems/infrastructure.html
[Hytera]: http://www.hytera-mobilfunk.com/dmr/
[RadioActivity]: http://www.radioactivity-tlc.com
[Multi-Mode Digital Voice Modem]: http://mmdvm.blogspot.com
[DVMEGA]: http://www.dvmega.auria.nl
[DV4mini]: http://www.g0hwc.com/dv4mini.html
[Tytera]: http://www.tyt888.com/
[Codec2]: http://www.rowetel.com/blog/?page_id=452
[Opus]: https://www.opus-codec.org/
[Brandmeister]: http://www.brandmeister.network/
[Go]: http://golang.org/
[Hose line]: http://hose.brandmeister.network/