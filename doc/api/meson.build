prg_install = find_program('install')

doxygen = find_program('doxygen', required : false)
if not doxygen.found()
	error('Program "doxygen" not found or not executable. Try building with -Ddocumentation=false')
endif
dot = find_program('dot', required : false)
if not dot.found()
	error('Program "dot" not found or not executable. Try building with -Ddocumentation=false')
endif

doxygen_version_cmd = run_command(doxygen.path(), '--version')
if doxygen_version_cmd.returncode() != 0
	error('Command "doxygen --version" failed.')
endif
doxygen_version = doxygen_version_cmd.stdout()
if doxygen_version.version_compare('< 1.8.3')
	error('doxygen needs to be at least version 1.8.3 (have @0@)'.format(doxygen_version))
endif
grep = find_program('grep')
dot_version_cmd = run_command(dot.path(), '-V')
if dot_version_cmd.returncode() != 0
	error('Command "dot -V" failed.')
endif
# dot -V output is (to stderr):
# 	dot - graphviz version 2.38.0 (20140413.2041)
dot_version = dot_version_cmd.stderr().split(' ')[4]
if dot_version.version_compare('< 2.26')
	error('Graphviz dot needs to be at least version 2.26 (have @0@)'.format(dot_version))
endif

doc_git_version = vcs_tag(command : ['git', 'log', '-1', '--format=%h'],
			  fallback : 'ERROR - unable to fetch git version',
			  input : 'git-version.dox',
			  output : 'git-version.dox',
			  replace_string: '__GIT_VERSION__')

mainpage = vcs_tag(command : ['git', 'log', '-1', '--format=%h'],
		 fallback : 'unknown',
		 input : 'mainpage.dox',
		 output : 'mainpage.dox',
		 replace_string: '__GIT_VERSION__')

src_doxygen = files(
	# source files
	join_paths(meson.source_root(), 'src', 'libinput.h'),
	# style files
	'style/header.html',
	'style/footer.html',
	'style/customdoxygen.css',
	'style/bootstrap.css',
	'style/libinputdoxygen.css',
)

doxyfiles = custom_target('doxyfiles',
			  input : src_doxygen,
			  output : 'doxyfiles',
			  command : [prg_install, '-t', '@OUTDIR@', '@INPUT@'],
			  build_by_default: true)

doc_config = configuration_data()
doc_config.set('PACKAGE_NAME', meson.project_name())
doc_config.set('PACKAGE_VERSION', meson.project_version())
doc_config.set('builddir', meson.current_build_dir())

doxyfile = configure_file(input : 'libinput.doxygen.in',
			  output : 'libinput.doxygen',
			  configuration : doc_config,
			  install : false)

custom_target('doxygen',
	      input : [ doxyfile, mainpage, doc_git_version] + src_doxygen,
	      output : [ '.' ],
	      command : [ doxygen, doxyfile ],
	      install : false,
	      depends: [doxyfiles, mainpage, doc_git_version],
	      build_by_default : true)