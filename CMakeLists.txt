# Copyright David Stone 2020.
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE_1_0.txt or copy at
# http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.28 FATAL_ERROR)

project(cmake_strict_defaults LANGUAGES CXX)

# For options that are necessary to use libraries: turn on latest language
# features
add_library(strict_defaults_interface INTERFACE)
# For options that are not necessary to use libraries, but are how I want to
# compile tests and source files
add_library(strict_defaults INTERFACE)

target_link_libraries(strict_defaults INTERFACE strict_defaults_interface)

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
	if(MSVC)
		# Sanitizers currently do not work very well on Windows
		set(use_sanitizers OFF)

		target_compile_options(strict_defaults INTERFACE
			"/MP"
		)
		target_link_options(strict_defaults INTERFACE
			"/WX"
		)
	else()
		target_link_options(strict_defaults INTERFACE
			"-Wl,--fatal-warnings"
			# TODO: Figure out how to make this work on Visual Studio
			$<$<CONFIG:Release>:-Wl,--thinlto-cache-dir=${CMAKE_BINARY_DIR}/lto-cache>
			$<$<CONFIG:Release>:-flto=thin>
		)
	endif()

	target_compile_options(strict_defaults INTERFACE
		"-Weverything"
		"-Werror"
		# Don't need compatibility with older standards
		"-Wno-c++20-compat"
		# Don't need compatibility with older standards
		"-Wno-c++20-extensions"
		# Don't need compatibility with older standards
		"-Wno-pre-c++26-compat"
		# Don't need compatibility with older standards
		"-Wno-c++98-compat"
		# Don't need compatibility with older standards
		"-Wno-c++98-compat-pedantic"
		# This warning tries to enforce a particular coding standard
		"-Wno-ctad-maybe-unsupported"
		# I cannot understand what makes this warning trigger, but it frequently
		# warns me about usages inside the standard library.
		"-Wno-deprecated-declarations"
		# Complains about comments like "//<N3757>" referring to an ISO paper number
		"-Wno-documentation"
		# There are too many cases of correct exact comparisons
		"-Wno-float-equal"
		# https://github.com/llvm/llvm-project/issues/49009
		"-Wno-for-loop-analysis"
		# -Wmissing-braces wants two sets of braces for std::array
		"-Wno-missing-braces"
		# No longer relevant with modules
		"-Wno-missing-prototypes"
		# No longer relevant with modules
		"-Wno-missing-variable-declarations"
		"-Wno-newline-eof"
		# The only problem is deleting through a pointer to base with a
		# non-virtual destructor. clang does warn about that case, so this
		# warning is only false positives.
		"-Wno-non-virtual-dtor"
		# https://github.com/llvm/llvm-project/issues/22816#issuecomment-980941775
		"-Wno-padded"
		# Warns about intentional copies in range-based for loops
		"-Wno-range-loop-analysis"
		# -Wswitch-default warns even if all enumerators are handled
		"-Wno-switch-default"
		# -Wswitch-enum warns even if you have default
		"-Wno-switch-enum"
		# https://github.com/llvm/llvm-project/issues/25458
		"-Wno-unneeded-member-function"
		# https://github.com/llvm/llvm-project/issues/25458
		"-Wno-unused-member-function"
		# https://github.com/llvm/llvm-project/issues/25458
		"-Wno-unused-function"
		# https://github.com/llvm/llvm-project/issues/25458
		"-Wno-unused-template"
		# Causes a conflict in module definitions. TODO: file bug
		"-Wno-unsafe-buffer-usage"
		# https://github.com/llvm/llvm-project/issues/85122
		"-Wno-unused-const-variable"
		# -Wweak-vtables is incompatible with header-only libraries. It also
		# becomes irrelevant after modules.
		"-Wno-weak-vtables"
		# https://github.com/llvm/llvm-project/issues/43670
		"-Wno-zero-as-null-pointer-constant"
		"-g"
		$<$<CONFIG:Release>:-flto=thin>
	)
	if (NOT CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 19)
		target_compile_options(strict_defaults INTERFACE
			# Do not warn about [[assume(condition)]]
			"-Wno-unknown-attributes"
		)
	endif()

elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	target_compile_options(strict_defaults_interface INTERFACE
		"-fconcepts"
	)

	target_compile_options(strict_defaults INTERFACE
		"-Wall"
		"-Wextra"
		"-Wpedantic"
		# Do not care about ABI differences
		# "-Wabi"
		"-Wcast-align"
		"-Wcast-qual"
		# https://gcc.gnu.org/bugzilla/show_bug.cgi?id=39170
		#"-Wconversion"
		# A reasonable code pattern is to use a private member function with
		# `if constexpr (requires { expr; }) { static_assert(expr); } }`
		# This might be the only member function of a class if it's a trait
		# class. Such a pattern triggers this warning.
		"-Wno-ctor-dtor-privacy"
		"-Wdisabled-optimization"
		"-Wdouble-promotion"
		# -Weffc++ includes a warning if all data members are not explicitly
		# initialized in the initializer list. I intentionally do not do this in
		# many cases. This would be more useful as a collection of warnings
		# like -Wall instead of a single warning on its own.
		# "-Weffc++"
		# There are too many cases of correct exact comparisons
		"-Wno-float-equal"
		"-Wformat=2"
		# Warns for `static_cast<decltype(x)>`
		"-Wno-ignored-qualifiers"
		"-Winit-self"
		"-Winvalid-pch"
		# -Wlogical-op warns for expressions that happen to be equal in a
		# template instantiation
		# "-Wlogical-op"
		"-Wmissing-declarations"
		# -Wmissing-format-attribute is not used because I do not use GNU
		# extensions. Same for -Wsuggest-attribute and several others.
		"-Wmissing-include-dirs"
		#"-Wnoexcept"
		"-Wno-non-template-friend"
		"-Wold-style-cast"
		"-Woverloaded-virtual"
		# https://gcc.gnu.org/bugzilla/show_bug.cgi?id=52981#c5
		"-Wno-padded"
		"-Wredundant-decls"
		"-Wshadow"
		# Warns for constant expressions that are guaranteed to be safe
		"-Wno-sign-conversion"
		"-Wsign-promo"
		# -Wsuggest-final-methods and -Wsuggest-final-types is a linker warning,
		# so it is not possible to disable it for boost and other third-party
		# libraries by saying they are system headers.
		# "-Wsuggest-final-methods"
		# "-Wsuggest-final-types"
		"-Wstrict-null-sentinel"
		# -Wstrict-overflow=2 warns about comparing two pointers
		"-Wstrict-overflow=1"
		# -Wswitch-default warns even if all enumerators are handled
		"-Wno-switch-default"
		# -Wswitch-enum warns even if there is a default
		"-Wno-switch-enum"
		"-Wtrampolines"
		"-Wundef"
		# -Wunsafe-loop-optimizations causes too many spurious warnings. It may
		# be useful to apply this one periodically and manually verify the
		# results. It generated this warning in my code when I looped over all
		# elements in a vector to apply a set of functions to them (using the
		# range-based for loop). It is also warning for the constructor of a
		# const array of const std::string where there is no loop in user code.
		# "-Wunsafe-loop-optimizations"
		# -Wunused-but-set-parameter does not properly account for
		# `if constexpr` branches.
		"-Wno-unused-but-set-parameter"
		# -Wunused-but-set-variable does not properly account for
		# `if constexpr` branches.
		"-Wno-unused-but-set-variable"
		# Incompatible with operator?: macro
		# "-Wuseless-cast"
		"-Wvector-operation-performance"
		# Warns on explicit template instantiations due to some bug
		"-Wno-unused-parameter"
		# -Wzero-as-null-pointer-constant does not work with the `operator<=>`
		# emulation
		# "-Wzero-as-null-pointer-constant"
		"-Werror"
		"-g"
		# $<$<CONFIG:Release>:-flto=4 -flto-odr-type-merging -fuse-linker-plugin>
	)
	target_link_options(strict_defaults INTERFACE
		"-Wl,--fatal-warnings"
	)
else()
	message(FATAL_ERROR "Unknown compiler ${CMAKE_CXX_COMPILER_ID}")
endif()

set(sanitizers "-fsanitize=address;-fsanitize=undefined")

if(DEFINED use_sanitizers)
	if(${use_sanitizers})
		target_compile_options(strict_defaults INTERFACE ${sanitizers})
		target_link_options(strict_defaults INTERFACE ${sanitizers})
	endif()
else()
	target_compile_options(strict_defaults INTERFACE
		$<$<CONFIG:Debug>:${sanitizers}>
	)
	target_link_options(strict_defaults INTERFACE
		$<$<CONFIG:Debug>:${sanitizers}>
	)
endif()
