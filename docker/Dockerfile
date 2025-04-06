ARG ROS_DISTRO=humble
FROM ros:${ROS_DISTRO}

ENV COLCON_WS=/root/ros2_ws

# set up thread number for building
RUN   if [ "x$(nproc)" = "x1" ] ; then export USE_PROC=1 ; \
      else export USE_PROC=$(($(nproc)/2)) ; fi

# Install binary dependencies with rosdep
WORKDIR $COLCON_WS
RUN   --mount=type=bind,source=./vins/package.xml,target=$COLCON_WS/vins/package.xml \
      --mount=type=bind,source=./loop_fusion,target=$COLCON_WS/loop_fusion \
      --mount=type=bind,source=./global_fusion,target=$COLCON_WS/global_fusion \
      --mount=type=bind,source=./camera_models,target=$COLCON_WS/camera_models \
      apt-get update && \
      rosdep update && \
      rosdep install --from-paths . --ignore-src -y && \
      rm -rf /var/lib/apt/lists/*

# Compile opencv from source because ubuntu doesn't yet distribute binaries that contain cudaoptflow.hpp
ARG OPENCV_VERSION=4.5.4
WORKDIR $COLCON_WS/src
RUN git clone --depth 1 --branch $OPENCV_VERSION https://github.com/opencv/opencv.git && \
    git clone --depth 1 --branch ${OPENCV_VERSION} https://github.com/opencv/opencv_contrib.git

WORKDIR $COLCON_WS/src/opencv
RUN cmake -B build \
    -D WITH_CUDA=ON \
    -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \
    -D OPENCV_ENABLE_NONFREE=ON \
    -D BUILD_opencv_cudacodec=ON \
    -D BUILD_opencv_cudaoptflow=ON
RUN cmake --build build --parallel $(nproc)
RUN cmake --install build
# Don't compile this again with colcon
RUN touch COLCON_IGNORE
WORKDIR $COLCON_WS/src/opencv_contrib
RUN touch COLCON_IGNORE

# Copy VINS-Fusion
WORKDIR $COLCON_WS
COPY ./ src/VINS-Fusion/

# Build VINS-Fusion
WORKDIR $COLCON_WS
ENV TERM=xterm
ENV PYTHONIOENCODING=UTF-8
SHELL ["/bin/bash", "-c"]
RUN source /opt/ros/${ROS_DISTRO}/setup.bash && \
      colcon build --cmake-args \
        -DCMAKE_BUILD_TYPE=Release
#       sed -i '/exec "$@"/i \
#             source "/root/catkin_ws/devel/setup.bash"' /ros_entrypoint.sh
