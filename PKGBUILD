# Maintainer: Alex S. <shantanna_at_hotmail_dot_com>
# Contributor: Jonathon Fernyhough <jonathon_at_manjaro_dot_org>
# Contributor: Andrew Shark <ashark#linuxcomp.ru>

# You'll need to download the package archive from
# https://www.blackmagicdesign.com/products/davinciresolve

# Hardware support is limited. Nvidia cards should work fine.
# If you're running a hybrid setup, try with primusrun/optirun.

pkgname=davinci-resolve
_pkgname=resolve
pkgver=15.2.2
pkgrel=2
pkgdesc='Professional A/V post-production software suite from Blackmagic Design'
arch=('x86_64')
url="https://www.blackmagicdesign.com/"
license=('Commercial')
depends=('glu' 'gtk2' 'gstreamer' 'libpng12' 'lib32-libpng12' 'ocl-icd' 'openssl-1.0'
         'opencl-driver' 'qt4' 'qt5-base' 'qt5-svg' 'qt5-webkit'
         'qt5-webengine' 'qt5-websockets')
         # TODO check that all of these needed. Also explore `ldd ./installer` and `ldd resolve`.
makedepends=('xdg-user-dirs' 'unzip' 'libisoburn')
options=('!strip') # What is this? Do we need it?
conflicts=('davinci-resolve-beta' 'davinci-resolve-studio' 'davinci-resolve-studio-beta')

# source=("DaVinci Resolve.desktop"
#           "start-resolve")
# sha256sums=('insert-sum-here'
#               'insert-sum-here')


# Order of the official installation procedure:
# You download .zip from blackmagicdesign site.
# Then you unpack it. Two files are inside: pdf and run.
# In pdf there is a link to dedicated centos iso installer with dr, which we may use for tests.
# .run file is actually AppImage. It starts AppRun file, which calls "installer" executable.
# For some reason they made it binary, so we cannot see what else it does. But here is order of its actions:
# - run scripts/pre_install.sh
# - copy all files except AppRun and .DirIcon to /opt/resolve
# - run scripts/post_install.sh
# So this pkgbuild is remake of these actions.

prepare() {
    _archive="DaVinci_Resolve_${pkgver}_Linux.zip"
    _archive_sha256sum='4330673cbe62f1ce2292d0357e20503233124bbb5a1b7752ce83b4befcf29497'
    
    DOWNLOADS_DIR=`xdg-user-dir DOWNLOAD`
    if [ ! -f ${srcdir}/${_archive} ]; then
        if [ -f $DOWNLOADS_DIR/${_archive} ]; then
            ln -sfn $DOWNLOADS_DIR/${_archive} ${srcdir}
        else
            msg2 "The package archive can be downloaded here: https://www.blackmagicdesign.com/products/davinciresolve/"
            msg2 "Please remember to put a downloaded package ${_archive} into the build directory or $DOWNLOADS_DIR"
            exit 1
        fi
    fi

    # check integrity
    if ! echo "${_archive_sha256sum} ${srcdir}/${_archive}" | sha256sum -c --quiet; then
        echo "Invalid checksum for ${_archive}"
        exit 1
    fi
    
    # extract package to srcdir
    unzip -o ${srcdir}/${_archive}
    
    msg2 "Extracting from bundle..."
    xorriso -osirrox on -indev DaVinci_Resolve_${pkgver}_Linux.run -extract / "${srcdir}/unpacked"
}

package() {

# pre_install.sh reimplementation start
    INSTALL_DIR="${pkgdir}/opt/resolve"
    USER_UID=0
    USER_HOME="${pkgdir}/root"

    mkdir -m 0775 -p "$INSTALL_DIR"
    chown $USER_UID "$INSTALL_DIR" -R
    # Installing Resolve as root may cause issues with file permissions when running Resolve as a non-root user. # from pdf manual. Check for correctness.
# pre_install.sh reimplementation end

# ./installer reimplementation start
    cp -rpT ${srcdir}/unpacked "${pkgdir}/opt/resolve"
    rm ${pkgdir}/opt/resolve/{AppRun,.DirIcon}
    rm ${pkgdir}/opt/resolve/graphics/watermark.png # clutter used by gui installer
# ./installer reimplementation end

# post_install.sh reimplementation start
    #COMMON_DATA_DIR=/var/davinci/resolve # never used
    RESOLVE_APP_NAME=com.blackmagicdesign.resolve
#     DBUS_SERVICE_DIR=/usr/share/dbus-1/services # never used
    
    FILES=(
            "${pkgdir}/opt/resolve/share/DaVinciResolve.desktop"
            "${pkgdir}/opt/resolve/share/DaVinciResolveInstaller.desktop"  
            "${pkgdir}/opt/resolve/share/DaVinciResolveCaptureLogs.desktop"
            "${pkgdir}/opt/resolve/share/DaVinciResolvePanelSetup.desktop"
            "${pkgdir}/opt/resolve/share/DaVinciResolve.directory"
            "${pkgdir}/opt/resolve/share/DaVinciResolve.menu"
           )
    ABSOLUTE_INSTALL_DIR="/opt/resolve"
    for file in "${FILES[@]}"; do
        sed -i -e "s:RESOLVE_INSTALL_LOCATION:${ABSOLUTE_INSTALL_DIR}:g" "$file" 
    done
    
    mkdir -p "${pkgdir}/usr/share/applications/"
    mkdir -p "${pkgdir}/usr/share/desktop-directories/"
    mkdir -p "${pkgdir}/etc/xdg/menus/applications-merged"
    
    mv ${pkgdir}/opt/resolve/share/DaVinciResolve.desktop ${pkgdir}/usr/share/applications/${RESOLVE_APP_NAME}.desktop
    rm ${pkgdir}/opt/resolve/share/DaVinciResolveInstaller.desktop
    rm ${pkgdir}/opt/resolve/graphics/DV_Uninstall.png
#   remove from /etc/xdg... <Filename>com.blackmagicdesign.resolve-Installer.desktop</Filename> # Do we even need that file?
    mv ${pkgdir}/opt/resolve/share/DaVinciResolveCaptureLogs.desktop ${pkgdir}/usr/share/applications/${RESOLVE_APP_NAME}-CaptureLogs.desktop
    mv ${pkgdir}/opt/resolve/share/DaVinciResolvePanelSetup.desktop ${pkgdir}/usr/share/applications/${RESOLVE_APP_NAME}-Panels.desktop
    mv ${pkgdir}/opt/resolve/share/DaVinciResolve.directory ${pkgdir}/usr/share/desktop-directories/${RESOLVE_APP_NAME}.directory
    mv ${pkgdir}/opt/resolve/share/DaVinciResolve.menu ${pkgdir}/etc/xdg/menus/applications-merged/${RESOLVE_APP_NAME}.menu
    
    echo "Installing Application icons"
    mkdir -p ${pkgdir}/usr/share/icons/hicolor/128x128
    mkdir -p ${pkgdir}/usr/share/mime/packages
    XDG_DATA_DIRS="${pkgdir}/usr/share"
    xdg-icon-resource install --size 128 "${INSTALL_DIR}/graphics/DV_Resolve.png" DaVinci-Resolve
    xdg-icon-resource install --size 128 "${INSTALL_DIR}/graphics/DV_ResolveProj.png" DaVinci-ResolveProj

    xdg-icon-resource install --size 128 --context mimetypes "${INSTALL_DIR}/graphics/DV_ResolveProj.png" application-x-resolveproj
#     xdg-mime install --novendor "${INSTALL_DIR}/share/resolve.xml" # not needed, handled by pacman
    cp "${INSTALL_DIR}/share/resolve.xml" ${pkgdir}/usr/share/mime/packages
    
    
    echo "Installing udev rules"
    mkdir -p ${pkgdir}/usr/lib/udev/rules.d/
    echo 'SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", ATTRS{idVendor}=="096e", MODE="0666"' > ${pkgdir}/usr/lib/udev/rules.d/75-davincipanel.rules
    echo 'SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", ATTRS{idVendor}=="1edb", MODE="0666"' > ${pkgdir}/usr/lib/udev/rules.d/75-sdx.rules

    
    echo "Setting up dvpanels"
    PANEL_DRIVER_DIR=${pkgdir}/usr/lib
    mkdir ${pkgdir}/usr/lib || :
    tar -xvf "${INSTALL_DIR}/share/panels/dvpanel-framework-linux-x86_64.tgz" libDaVinciPanelAPI.so # I have not used a lib folder from archive
    mv libDaVinciPanelAPI.so "${PANEL_DRIVER_DIR}"
    rm "${INSTALL_DIR}/share/panels/dvpanel-framework-linux-x86_64.tgz"
    
    # From original:
    # # We install the default app associations here, because xdg-mime is buggy in centos6
    # install_default_application ${RESOLVE_APP_NAME}.desktop application/x-resolveproj
    # # We install the default application system-wide. Per user default-app preferences are buggy for Centos6
    # install_default_application()
    # {
    # $1 is .desktop
    # $2 is mime/type
    # DEFAULT_MIME_FILE="/usr/share/applications/defaults.list"
    #     grep -v "$2=" $DEFAULT_MIME_FILE > ${DEFAULT_MIME_FILE}.new 2> /dev/null
    #     if ! grep "[Default Applications]" ${DEFAULT_MIME_FILE}.new > /dev/null; then
    #     echo "[Default Applications]" >> ${DEFAULT_MIME_FILE}.new
    #     fi
    #     echo $2=$1 >> ${DEFAULT_MIME_FILE}.new
    #     mv ${DEFAULT_MIME_FILE}.new $DEFAULT_MIME_FILE
    # }
    # Here I repeat this, but then package owns this file, which is not correct. Do we even need it?
    echo "[Default Applications]" > "${pkgdir}/usr/share/applications/defaults.list"
    echo "application/x-resolveproj=com.blackmagicdesign.resolve.desktop" >> "${pkgdir}/usr/share/applications/defaults.list"
    # XDG_DATA_DIRS="${pkgdir}/usr/share" xdg-desktop-menu install --noupdate --novendor misc/tcharmap.desktop # this is from tcharmap-git package
    
    
    LIST_OF_USER_FILES=(
        "configs"
        "easyDCP"
        "logs"
        "Developer"
        "DolbyVision"
        "LUT"
        ".LUT"
        ".license"
        ".crashreport"
        "Resolve Disk Database"
        "Fairlight"
        "Media"
        )

    for file in "${LIST_OF_USER_FILES[@]}"; do
        mkdir -p -m 0775 "$INSTALL_DIR/$file"
        chown $USER_UID "$INSTALL_DIR/$file"
    done

    backup=( opt/resolve/configs/config.dat
             opt/resolve/configs/log-conf.xml
             opt/resolve/DolbyVision/config.bin )
    mv "${INSTALL_DIR}/share/default-config-linux.dat" "${INSTALL_DIR}/configs/config.dat"
    mv "${INSTALL_DIR}/share/log-conf.xml" "${INSTALL_DIR}/configs/log-conf.xml"
    mv "${INSTALL_DIR}/share/default_cm_config.bin" "${INSTALL_DIR}/DolbyVision/config.bin"

    # Can we somehow handle user permissions correctly for these files?
    
# post_install.sh reimplementation end

    
# export BMD_PLUGIN_PATH=$CURRENT_DIR/libs/plugins # from AppRun, used by ./installer. Do not know its effect.

# Do we still need this?
# msg2 "Creating launchers..."
# cd "${srcdir}" || exit
# cat > "${srcdir}/DaVinci Resolve.desktop" << EOF
# #!/usr/bin/env xdg-open
# [Desktop Entry]
# Type=Application
# Name=DaVinci Resolve
# Comment=Professional non-linear editing
# Exec=/opt/${_pkgname}/bin/start-resolve
# Icon=/opt/${_pkgname}/rsf/DV_Resolve.png
# Terminal=false
# Categories=Multimedia;AudioVideo;Application;
# EOF
# install -Dm644 DaVinci\ Resolve.desktop "${pkgdir}/usr/share/applications/DaVinci Resolve.desktop"
# 
# cat > "${srcdir}/start-resolve" << EOF
# #!/bin/sh
# mkdir -p /tmp/${_pkgname}/{logs,GPUCache}
# cd /opt/${_pkgname}
# exec bin/resolve "\$@"
# EOF
# install -Dm755 start-resolve "${pkgdir}/opt/${_pkgname}/bin/start-resolve"
# 
# msg2 "Making sure file ownership is 'correct'..."
# chown -R root:root "${pkgdir}/opt"
# chmod 0777 "${pkgdir}/opt/${_pkgname}/configs"
# chmod 0777 "${pkgdir}/opt/${_pkgname}/Media"
# 
# msg2 "Any final tweaks..."
# ln -s "/tmp/${_pkgname}/logs" "${pkgdir}/opt/${_pkgname}/logs"
# ln -s "/tmp/${_pkgname}/GPUCache" "${pkgdir}/opt/${_pkgname}/GPUCache"
# 
    msg2 "Done!"
}
