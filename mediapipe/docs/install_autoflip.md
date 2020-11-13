# AutoFlip Linux installation notes

This file is an appendix to [install.md](/mediapipe_fork/mediapipe/docs/install.md), correcting the installation procedure for Linux Desktops.

## Installing OpenCV 3.4.9 for usage with Google's Mediapipe AutoFlip

- Uninstall OpenCV4 by removing files that match. Note - more advanced users that know how to avoid conflicts across installs may wish to skip this step and implement their own solution for multiple installs

  ```bash
  sudo rm -rf /usr/local/lib/cmake/opencv4
  sudo rm -rf /usr/local/lib/opencv4
  sudo rm -rf /usr/local/share/licenses/opencv4
  sudo rm -rf /usr/local/share/opencv4
  ```

- In `~/`, download OpenCV and OpenCV_Contrib source files from the relevant release pages:

  ```bash
  wget https://github.com/opencv/opencv/archive/3.4.9.zip
  wget https://github.com/opencv/opencv_contrib/archive/3.4.9.zip
  ```

- Unzip the resulting files. Feel free to `rm` the original `.zip` files. There should now be two new folders in your home directory: `opencv-3.4.9/` and `opencv_contrib-3.4.9/`
- In `~/opencv-3.4.9/` run

  ```bash
  mkdir build
  cd build
  ```

- In `~/opencv-3.4.9/build/` run

  ```bash
  cmake .. -DCMAKE_BUILD_TYPE=RELEASE -DCMAKE_INSTALL_PREFIX=/usr/local \
          -DBUILD_TESTS=OFF -DBUILD_PERF_TESTS=OFF -DBUILD_opencv_ts=OFF \
          -DOPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-3.4.9/modules \
          -DBUILD_opencv_aruco=OFF -DBUILD_opencv_bgsegm=OFF -DBUILD_opencv_bioinspired=OFF \
          -DBUILD_opencv_ccalib=OFF -DBUILD_opencv_datasets=OFF -DBUILD_opencv_dnn=OFF \
          -DBUILD_opencv_dnn_objdetect=OFF -DBUILD_opencv_dpm=OFF -DBUILD_opencv_face=OFF \
          -DBUILD_opencv_fuzzy=OFF -DBUILD_opencv_hfs=OFF -DBUILD_opencv_img_hash=OFF \
          -DBUILD_opencv_js=OFF -DBUILD_opencv_line_descriptor=OFF -DBUILD_opencv_phase_unwrapping=OFF \
          -DBUILD_opencv_plot=OFF -DBUILD_opencv_quality=OFF -DBUILD_opencv_reg=OFF \
          -DBUILD_opencv_rgbd=OFF -DBUILD_opencv_saliency=OFF -DBUILD_opencv_shape=OFF \
          -DBUILD_opencv_structured_light=OFF -DBUILD_opencv_surface_matching=OFF \
          -DBUILD_opencv_world=OFF -DBUILD_opencv_xobjdetect=OFF -DBUILD_opencv_xphoto=OFF
  ```

- In `~/opencv-3.4.9/build/` run

  ```bash
  make -j 16 # this will take a while
  sudo make install
  ```

  OpenCV has been built. You can find the header files and libraries in `/usr/local/include/` and `/usr/local/lib`

- in your `mediapipe/` directory, edit `WORKSPACE` so that you have

  ```plain
  new_local_repository(
      name = "linux_opencv",
      build_file = "@//third_party:opencv_linux.BUILD",
      path = "/usr/local",
  )
  ```

- Likewise, edit `mediapipe/third_party/opencv_linux.BUILD` so that you have

  ```plain
  cc_library(
      name = "opencv",
      srcs = glob(
          [
              "lib/libopencv_core.so",
              "lib/libopencv_highgui.so",
              "lib/libopencv_imgcodecs.so",
              "lib/libopencv_imgproc.so",
              "lib/libopencv_video.so",
              "lib/libopencv_videoio.so",
          ],
      ),
      hdrs = glob(["local/include/opencv/*.h*"]),
      includes = ["local/include/opencv/"],
      linkstatic = 1,
      visibility = ["//visibility:public"],
  )
  ```

- Run

  ```bash
  sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
  sudo ldconfig
  ```
