find_package(benchmark CONFIG QUIET)

if(NOT benchmark_FOUND)
  message(STATUS "google-benchmark not found; skipping benchmark targets. "
                 "Install via vcpkg (benchmark) or `brew install google-benchmark` to enable.")
  return()
endif()

function(spp_add_bench name src)
  add_executable(${name} ${src})
  target_link_libraries(${name} PRIVATE spp_core benchmark::benchmark benchmark::benchmark_main)
  target_include_directories(${name} PRIVATE ${CMAKE_SOURCE_DIR}/src)
  spp_apply_warnings(${name})
  # spp_core is built with -fsanitize=address,undefined when SPP_ENABLE_SANITIZERS
  # is on. Without applying the same flags here the bench binaries link against
  # the instrumented static lib but skip the ASan/UBSan runtime, so on Linux
  # they fail to link with undefined references to __asan_report_* and
  # __ubsan_handle_* — every other target category (tools, tests, examples,
  # spp_core itself) already applies sanitizers; benches were the gap.
  spp_apply_sanitizers(${name})
endfunction()

spp_add_bench(posting_list_bench       posting_list_bench.cpp)
spp_add_bench(query_latency_bench      query_latency_bench.cpp)
spp_add_bench(indexing_throughput_bench indexing_throughput_bench.cpp)
spp_add_bench(rerank_latency_bench     rerank_latency_bench.cpp)
