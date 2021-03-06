FROM registry.anybox.cloud/anybox/odoo-docker/ocb:12.0

LABEL maintainer="Anybox"
ENV VERSION 12.0
USER root

RUN set -x; \
    # build deps for addons
    apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        libldap2-dev \ 
        libpq-dev \
        libsasl2-dev \
        python3-dev \
        sudo \
    ### DOWNLOAD ADDONS
    && cd /srv/extra-addons \
    && for repo in \
        OCA/contract \
        OCA/crm \
        OCA/knowledge \
        OCA/l10n-france \
        OCA/partner-contact \
        OCA/project \
        OCA/server-brand \
        OCA/server-tools \
        OCA/web \
        OCA/website \
        Openworx/backend_theme \
    ; do curl -o addons.zip -sSL https://github.com/$repo/archive/$VERSION.zip \
        && unzip -q addons.zip \
        && rm -rf ${repo#*/}-$VERSION/setup \
        && mv -f ${repo#*/}-$VERSION/* . \
        && rm -rf ${repo#*/}-$VERSION \
    ; done \
    && chown -R odoo:nogroup . \
    # detect, fix and install python deps of odoo addons (functional style)
    && for m in `python -c "\
from openerp.modules import module; \
from itertools import chain; \
print(' '.join(set(chain(*[\
    module.load_information_from_description_file(m).get(\
    'external_dependencies',{'python':[]})['python'] \
    for m in module.get_modules()]))).replace( \
    'ldap', 'python-ldap').replace( \
    'dateutil', 'python-dateutil').replace( \
    'lxml.html', 'lxml').replace( \
    'usb.core', 'pyusb').replace( \
    'jwt', 'python-jwt').replace( \
    'stdnum', 'python-stdnum'))"`; \
        do pip3 install --no-cache-dir $m; done \
    && pip3 install --no-cache-dir num2words phonenumbers simplejson \
    # cleanup
    && rm -rf .gitignore README.md .travis.yml *zip \
    && SUDO_FORCE_REMOVE=yes apt-get purge -y --auto-remove \
        -o APT::AutoRemove::RecommendsImportant=false \
        -o APT::AutoRemove::SuggestsImportant=false \
        build-essential \
        libldap2-dev \ 
        libpq-dev \
        libsasl2-dev \
        python3-dev \
        sudo \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
            
USER odoo            
VOLUME /var/lib/odoo
COPY odoo.conf /etc/odoo/

