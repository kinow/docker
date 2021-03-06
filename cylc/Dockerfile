# The MIT License (MIT)
# 
# Copyright (c) 2016 Bruno P. Kinoshita
# 
# Permission is hereby granted, free of charge, to any person obtaining a copy of this software and 
# associated documentation files (the "Software"), to deal in the Software without restriction, including 
# without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell 
# copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the 
# following conditions:
# 
# The above copyright notice and this permission notice shall be included in all copies or substantial 
# portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT 
# LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO 
# EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER 
# IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR 
# THE USE OR OTHER DEALINGS IN THE SOFTWARE.

FROM ubuntu:14.04
MAINTAINER Bruno P. Kinoshita <brunodepaulak@yahoo.com.br>

# A docker container with cylc installed.

RUN apt-get -y update && \
	apt-get -y upgrade && \
	apt-get -y install \
	git \
	python python-dev python-pip \
	pkg-config \
	graphviz graphviz-dev libgraphviz-dev \
	sqlite \
	python-gtk2-dev \
	python-pygraphviz

RUN cd /opt && \
	git clone https://github.com/cylc/cylc.git && \
	cd cylc && \
	git checkout 6.8.1

# pygraphviz seems broken on ubuntu, when installed via pip
# see https://github.com/pygraphviz/pygraphviz/issues/71
RUN cd /opt/cylc && \
	pip install "Pyro>=3.10,<4.0" && \
	pip install Jinja2 && \
	pip install pygraphviz --install-option="--include-path=/usr/include/graphviz" --install-option="--library-path=/usr/lib/graphviz/"

RUN cd /opt/cylc/ && \
	./bin/cylc check-software && \
	./bin/cylc version
