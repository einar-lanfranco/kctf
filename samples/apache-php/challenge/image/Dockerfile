# Copyright 2020 Google LLC
# 
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#     https://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM kctf-nsjail

COPY flag /chroot/

VOLUME /tmp
VOLUME /var/log/apache2
VOLUME /var/run/apache2

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends tzdata \
    && ln -fs /usr/share/zoneinfo/Europe/Berlin /etc/localtime \
    && dpkg-reconfigure --frontend noninteractive tzdata \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -yq --no-install-recommends apache2 \
    && rm -rf /var/lib/apt/lists/*
RUN chroot /chroot apt-get update \
    && chroot /chroot apt-get install -yq --no-install-recommends php-cgi \
    && rm -rf /chroot/var/lib/apt/lists/*

RUN service apache2 start

RUN ln -s /etc/apache2/mods-available/cgi.load /etc/apache2/mods-enabled/cgi.load
RUN ln -s /etc/apache2/mods-available/actions.load /etc/apache2/mods-enabled/actions.load

COPY apache2-nsjail-php.conf /etc/apache2/conf-enabled/nsjail-php.conf

RUN rm -rf /var/www
RUN ln -s /chroot/var/www /var/www

RUN rm -rf /chroot/var/lib/php/sessions
RUN chroot /chroot rm -rf /var/lib/php/sessions
RUN chroot /chroot mkdir -p /mnt/disks/sessions /mnt/disks/uploads
RUN chroot /chroot ln -s /mnt/disks/sessions /var/lib/php/sessions

COPY html/ /chroot/var/www/html
COPY cgi-bin/ /usr/lib/cgi-bin
COPY root/ /root
RUN chmod 0755 /root

RUN chmod 0755 /chroot/var/www/html /usr/lib/cgi-bin/
RUN chmod 0755 /chroot/var/www/html/*.php /usr/lib/cgi-bin/nsjail-php-cgi /root/launch-apache

CMD kctf_setup && /root/launch-apache
