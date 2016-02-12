#=============================================================================
#   CMake build system files
#
#   Copyright (c) 2015 pocl developers
#
#   Permission is hereby granted, free of charge, to any person obtaining a copy
#   of this software and associated documentation files (the "Software"), to deal
#   in the Software without restriction, including without limitation the rights
#   to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#   copies of the Software, and to permit persons to whom the Software is
#   furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included in
#   all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#   AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#   OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
#   THE SOFTWARE.
#
#=============================================================================


if(CMAKE_HOST_SYSTEM_NAME STREQUAL "Linux" AND X86_64)

  set(TS_NAME AMDSDK3.0)
  set(TS_BASEDIR "${TESTSUITE_BASEDIR}/${TS_NAME}")
  set(TS_SRCDIR "${TESTSUITE_SOURCE_BASEDIR}/${TS_NAME}")
  set(AMD_APP_SDK_TGZ "${TS_SRCDIR}/AMD-APP-SDK-v3.0.130.135-GA-linux64.sh")

  if(EXISTS "${AMD_APP_SDK_TGZ}")

    message(STATUS "Enabling testsuite ${TS_NAME}")
    list(APPEND ACTUALLY_ENABLED_TESTSUITES "${TS_NAME}")
    set(ACTUALLY_ENABLED_TESTSUITES ${ACTUALLY_ENABLED_TESTSUITES} PARENT_SCOPE)

    ExternalProject_Add(
      ${TS_NAME}
      PREFIX "${TS_BASEDIR}"
      DOWNLOAD_COMMAND "/bin/true"
      PATCH_COMMAND /bin/sh "${AMD_APP_SDK_TGZ}" --noexec --keep --target AMD-APP-SDK-3.0 &&
             patch -p1 -i ${CMAKE_CURRENT_SOURCE_DIR}/amdsdk3_0.patch
      CONFIGURE_COMMAND "/bin/true"
      BUILD_IN_SOURCE 1
      BUILD_COMMAND "/bin/true"
      INSTALL_COMMAND "/bin/true"
    )

    set_target_properties(${TS_NAME} PROPERTIES EXCLUDE_FROM_ALL TRUE)
    add_dependencies(prepare_examples ${TS_NAME})

    set(AMD_SAMPLES
    AdvancedConvolution
    BasicDebug
    BinomialOption
    BinomialOptionMultiGPU
    BitonicSort
    BlackScholes
    BlackScholesDP
    BoxFilter
    BufferImageInterop
    BuiltInScan
    CalcPie
    ConcurrentKernel
    DCT
    DeviceFission
    DeviceFission11Ext
    DwtHaar1D
    FastWalshTransform
    FloydWarshall
    GaussianNoise
    HDRToneMapping
    HelloWorld
    Histogram
    HistogramAtomics
    ImageBinarization
    ImageOverlap
    KernelLaunch
    LUDecomposition
    MatrixMulImage
    MatrixMultiplication
    MatrixTranspose
    MemoryModel
    MonteCarloAsian
    MonteCarloAsianMultiGPU
    NBody
    PrefixSum
    QuasiRandomSequence
    RadixSort
    RecursiveGaussian
    Reduction
    ScanLargeArrays
    SimpleConvolution
    SimpleImage
    SimpleMultiDevice
    SobelFilter
    StringSearch
    Template
    UnsharpMask
    URNG)

    # freeze (requires async queue): FineGrainSVM
    # disabled c++ tests: DwtHaar1DCPPKernel EigenValue
    #     IntroStaticCPPKernel FFT MatrixMultiplicationCPPKernel
    #     MersenneTwister SoAversusAoS TransferOverlapCPP
    # disabled but fixable: BuiltInScan CalcPie CplusplusWrapper
    #     DeviceFission FluidSimulation2D GaussianNoise HDRToneMapping
    #     HeatPDE ImageBandwidth KernelLaunch MatrixMulDouble
    #     MonteCarloAsianDP SobelFilterImage UnsharpMask DynamicOpenCLDetection
    #     FineGrainSVMCAS RecursiveGaussian_ProgramScope SobelFilterImage
    #     SVMAtomicsBinaryTreeInsert SVMBinaryTreeSearch
    # disabled tests: AsyncDataTransfer AtomicCounters BasicDebug
    #     DeviceFission11Ext RangeMinimumQuery SimpleDepthImage
    #     SimpleGenericAddressSpace
    # disabled graphics tests: Mandelbrot  KmeansAutoclustering
    #     GaussianNoiseGL SimpleGL BoxFilterGL URNGNoiseGL
    # very slow: BufferBandwidth LDSBandwidth ConstantBandwidth
    #     MemoryOptimizations TransferOverlap
    # disabled - requires dev queue: BinarySearchDeviceSideEnqueue
    #     DeviceEnqueueBFS ExtractPrimes GlobalMemoryBandwidth
    #     RegionGrowingSegmentation SimpleSPIR
    # disabled - requires pipe: PipeProducerConsumerKernels SimplePipe


    foreach(SAMPLE IN LISTS AMD_SAMPLES)
      add_test(NAME "AMD_30_${SAMPLE}"
        COMMAND "${TS_BASEDIR}/src/${TS_NAME}/AMD-APP-SDK-3.0/samples/opencl/bin/x86_64/${SAMPLE}"
        WORKING_DIRECTORY "${TS_BASEDIR}/src/${TS_NAME}/AMD-APP-SDK-3.0/samples/opencl/bin/x86_64")
      set_tests_properties("AMD_30_${SAMPLE}" PROPERTIES LABELS "amdsdk_30")
    endforeach()

    # AMD_30_HeatPDE
    # AMD_30_FineGrainSVMCAS
    set_tests_properties(
        AMD_30_BinomialOption AMD_30_CalcPie AMD_30_DCT AMD_30_BlackScholes
        AMD_30_FastWalshTransform AMD_30_FloydWarshall AMD_30_HelloWorld
        AMD_30_Histogram AMD_30_MatrixMultiplication AMD_30_MatrixTranspose
        AMD_30_PrefixSum AMD_30_QuasiRandomSequence AMD_30_ScanLargeArrays
        AMD_30_SimpleConvolution AMD_30_URNG
      PROPERTIES
        LABELS "hsa")

  else()
    message(WARNING "Disabling testsuite ${TS_NAME} - tarball not found")
  endif()

else()
  message(warning "AMD APP SDK testsuite is only enabled for x86_64 Linux systems currently")
endif()