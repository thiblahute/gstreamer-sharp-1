project('gstreamer-sharp', ['cs', 'c'], version: '1.13.0.1',
    meson_version: '>0.42', license: 'LGPL')

if host_machine.system() == 'osx'
  lib_prefix=''
  lib_suffix='.dylib'
else
  lib_prefix='.so'
  lib_suffix=''
endif

gtk_sharp_required_version = '>=3.22.6'

csc = meson.get_compiler('cs')
if csc.get_id() == 'mono'
    add_project_arguments('-keyfile:' + join_paths(meson.current_source_dir(), meson.project_name() + '.snk'),
        language: ['cs'])
endif


subproject('gtk-sharp', default_options: ['install=false'])
gtk_sharp = subproject('gtk-sharp')

glib_sharp_dep = gtk_sharp.get_variable('glib_sharp_dep')
glib_sharp = gtk_sharp.get_variable('glib_sharp')

gio_sharp_dep = gtk_sharp.get_variable('gio_sharp_dep')
gio_sharp = gtk_sharp.get_variable('gio_sharp')

gapi_fixup = gtk_sharp.get_variable('gapi_fixup')
gapi_codegen = gtk_sharp.get_variable('gapi_codegen')

glib_api_includes = gtk_sharp.get_variable('glib_api_includes')
gio_api_includes = gtk_sharp.get_variable('gio_api_includes')
has_gtk = gtk_sharp.get_variable('has_gtk')
if has_gtk
  gtk_sharp_dep = [gtk_sharp.get_variable('gtk_sharp_dep'),
    gtk_sharp.get_variable('atk_sharp_dep')]
else
  gtk_sharp_dep = dependency('gtk-sharp-3.0', version: gtk_sharp_required_version,
      required: false)
endif
codegen_dependencies = [gapi_codegen, gapi_fixup, glib_sharp, gio_sharp]
gapi_fixup = gapi_fixup.full_path()
gapi_codegen = gapi_codegen.full_path()

gacutil = find_program('gacutil')
generate_api = find_program('generate_code.py')
nuget = find_program('nuget.py')

# TODO Handle monodoc

apiversion = '1.0'
gst_required_version = '>=@0@'.format(meson.project_version())
gst_deps = []

foreach dep: [
  ['gstreamer', ['gstreamer', 'gst_dep']],
  ['gstreamer-app', ['gst-plugins-base', 'app_dep']],
  ['gstreamer-audio', ['gst-plugins-base', 'audio_dep']],
  ['gstreamer-base', ['gst-plugins-base', 'audio_dep']],
  ['gstreamer-controller', ['gstreamer', 'gst_controller_dep']],
  ['gstreamer-fft', ['gst-plugins-base', 'fft_dep']],
  ['gstreamer-net', ['gstreamer', 'gst_net_dep']],
  ['gstreamer-pbutils', ['gst-plugins-base', 'pbutils_dep']],
  ['gstreamer-riff', ['gst-plugins-base', 'riff_dep']],
  ['gstreamer-rtp', ['gst-plugins-base', 'rtp_dep']],
  ['gstreamer-rtsp', ['gst-plugins-base', 'rtsp_dep']],
  ['gstreamer-sdp', ['gst-plugins-base', 'sdp_dep']],
  ['gstreamer-tag', ['gst-plugins-base', 'tag_dep']],
  ['gstreamer-video', ['gst-plugins-base', 'video_dep']],]

  gst_deps += [dependency(dep.get(0) + '-' + apiversion, version: gst_required_version,
    fallback: dep.get(1))]
endforeach

ges_dep = dependency('gst-editing-services-' + apiversion, version: gst_required_version,
    fallback: ['gst-editing-services', 'ges_dep'])

subdir('sources')
if ges_dep.found()
  subdir('ges')
  custom_target('GESSharp-nuget',
      command: [nuget, '--package-name', 'GESSharp',
          '--assembly', ges_sharp.full_path(),
          '--assembly', gst_sharp.full_path(),
          '--assembly', gio_sharp.full_path(),
          '--assembly', glib_sharp.full_path(),
          '--dependency=GstSharp:' + meson.project_version(),
          '--project-url', 'https://gstreamer.freedesktop.org/modules/gstreamer-sharp.html',
          '--icon-url', 'https://gstreamer.freedesktop.org/images/header-logo-top.png',
          '--license-url', 'https://cgit.freedesktop.org/gstreamer/gstreamer-sharp/plain/COPYING',
          '--version', meson.project_version(),
          '--tags', 'GStreamer GES GstSharp bindings',
          '--builddir', meson.current_build_dir(),
      ],
      output: 'GESSharp.' + meson.project_version() + '.nupkg'
  )
endif

custom_target('GstSharp-nuget',
    command: [nuget, '--package-name', 'GstSharp',
        '--assembly', gst_sharp.full_path(),
        '--assembly', gio_sharp.full_path(),
        '--assembly', glib_sharp.full_path(),
        '--project-url', 'https://gstreamer.freedesktop.org/modules/gstreamer-sharp.html',
        '--icon-url', 'https://gstreamer.freedesktop.org/images/header-logo-top.png',
        '--license-url', 'https://cgit.freedesktop.org/gstreamer/gstreamer-sharp/plain/COPYING',
        '--version', meson.project_version(),
        '--tags', 'GStreamer GstSharp bindings',
        '--builddir', meson.current_build_dir(),
    ],
    output: 'GstSharp.' + meson.project_version() + '.nupkg',
)

subdir('samples')

bindinate = find_program('bindinate', required: false)

if bindinate.found()
  run_target('update-bindings',
    command: [bindinate,
      '--name=gstreamer', '--regenerate=true',
      '--merge-with=GstApp-1.0,GstAudio-1.0,GstBase-1.0,GstController-1.0,GstFft-1.0,GstNet-1.0,GstPbutils-1.0,GstRiff-1.0,GstRtp-1.0,GstRtsp-1.0,GstSdp-1.0,GstTag-1.0,GstVideo-1.0',
      '--gir=Gst-1.0']
    )
  python3 = import('python3').find_python()
  run_target('update-ges-bindings',
    command: ['sh', '-c',
      '''bindinate --name=gst-editing-services --regenerate=true --gir=GES-1.0 && mv @0@/sources/gst-editing-services-sharp-api.raw @0@/ges/gst-editing-services-api.raw '''.format(
        meson.current_source_dir())
    ]
  )
endif
