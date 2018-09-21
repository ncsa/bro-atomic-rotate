bro-atomic-rotate
=================

Installation:

    cp zz_rotate /usr/local/bro/share/broctl/scripts/postprocessors/
    cp bro-atomic-rotate /usr/local/bin
    cp bro-atomic-rotate.service /etc/systemd/system
    systemctl enable bro-atomic-rotate
    systemctl start bro-atomic-rotate
