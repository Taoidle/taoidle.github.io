---
title: Python图像处理 —— 相机标定、畸变矫正
tags:
  - opencv
  - python
  - 原创教程
  - 图像处理
  - 教程
id: '342'
categories:
  - - 计算机视觉
date: 2020-09-02 17:12:15
---

## 实验数据

*   标定图（黑白棋盘图）若干（不少于10张拍摄不同角度的图片，焦距不变）

## 过程

*   每张图片提取角点信息
*   每张图片提取亚像素角点信息
*   相机标定
*   畸变矫正

## 内参数矩阵、畸变系数

*   这里保存为json文件

```json
{
  "mtx": [
    [
      3096.855023641478,
      0.0,
      1029.4641305618723
    ],
    [
      0.0,
      3091.0248015384295,
      333.24889640636945
    ],
    [
      0.0,
      0.0,
      1.0
    ]
  ],
  "dist": [
    [
      -0.102230276138444,
      -6.080581251041141,
      0.013564529498038387,
      -0.0027957254737596593,
      39.277124343630284
    ]
  ]
}

```

## 矫正前图像

![](/assets/images/wp_editor_md_46c192be1a69cf0a3d0a1e2d6e95eafb.jpg)

## 矫正后图像

![](/assets/images/wp_editor_md_f77ee67a039c3902f6253c5f1cab1f8a.jpg)

## 代码

*   包括图片矫正、视频矫正

```python
import os
import cv2
import glob
import json
import numpy as np


class CameraCalibration:

    def calibration_init(self, width, height, pic_path, save_parameters=True):
        # 亚像素角点的参数
        criteria = (cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 30, 0.001)

        # 标定板角点的位置
        objp = np.zeros((width * height, 3), np.float32)
        objp[:, :2] = np.mgrid[0:width, 0:height].T.reshape(-1, 2)

        # 存储点
        obj_points, img_points = [], []

        images = glob.glob(pic_path)
        for i in images:
            img = cv2.imread(i)
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
            # 棋盘格角点
            ret, corners = cv2.findChessboardCorners(gray, (width, height), None)
            # 存储角点
            if ret:
                cv2.cornerSubPix(gray, corners, (11, 11), (-1, -1), criteria)
                obj_points.append(objp)
                img_points.append(corners)

        # 标定
        ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(obj_points, img_points, gray.shape[::-1], None, None)

        if save_parameters:
            self.__parameters_init(mtx, dist)
        else:
            print("ret:", ret)
            print("mtx:\n", mtx)  # 内参数矩阵
            print("dist:\n", dist)  # 畸变系数   distortion cofficients = (k_1,k_2,p_1,p_2,k_3)
            print("rvecs:\n", rvecs)  # 旋转向量  # 外参数
            print("tvecs:\n", tvecs)  # 平移向量  # 外参数

    def __parameters_init(self, mtx, dist):
        parameters_path = './parameters.json'
        if not os.path.exists(parameters_path):
            os.system(r'touch %s' % parameters_path)
            self.__write_parameters(mtx, dist)
        else:
            self.__write_parameters(mtx, dist)

    @staticmethod
    def __write_parameters(mtx, dist):
        with open("./parameters.json", "w", encoding='utf-8') as f:
            content = {"mtx": mtx.tolist(), "dist": dist.tolist()}
            f.write(json.dumps(content, ensure_ascii=False))
        f.close()


class CameraCorrect:

    def __init__(self):
        self.__correct_init()

    # 畸变参数初始化
    def __correct_init(self, parameters_file_path: str = "./parameters.json"):
        if not os.path.exists(parameters_file_path):
            pass
        else:
            with open(parameters_file_path, "r", encoding='utf-8') as f:
                json_data = json.load(f)
                self.mtx = np.array(json_data["mtx"])
                self.dist = np.array(json_data["dist"])

    # 图片畸变校准
    def correct_pic(self, origin_path: str, target_path: str = "./", file_name: str = "cor_pic",
                    file_format: str = "png"):
        img = cv2.imread(origin_path)
        pic_height, pic_width = img.shape[:2]
        file_format = self.__calibration_pic_format(file_format.lower())
        cv2.imwrite(target_path + file_name + "." + file_format,
                    self.__distortion_correction(img, pic_width, pic_height))

    # 视频畸变校准(直接输出视频)
    def correct_vid2vid(self, origin_path: str, target_path: str = "./", file_name: str = "cor_vid",
                        file_format: str = "mp4", file_coding: str = "mp4v"):
        cap = cv2.VideoCapture(origin_path)
        # 获取视频参数
        fps, count, vid_width, vid_height = self.__get_capture_attribute(cap)
        # 格式编码校对
        if self.__calibration_format_coding(file_format, file_coding) is False:
            file_format, file_coding = "mp4", "mp4v"
        out = cv2.VideoWriter(target_path + file_name + "." + file_format,
                              cv2.VideoWriter_fourcc(*self.__get_vid_coding(file_coding)), fps,
                              (vid_width, vid_height))
        for i in range(count):
            ret, img = cap.read()
            if ret:
                img = cv2.resize(img, (vid_width, vid_height))
                dst = self.__distortion_correction(img, vid_width, vid_height)
                out.write(dst)
        cap.release()
        out.release()

    # 视频畸变校准(校准抽取所有帧)
    def correct_vid2pic(self, origin_path: str, target_path: str = "./", file_name: str = "cor_pic",
                        file_format: str = "png"):
        file_format = self.__calibration_pic_format(file_format.lower())
        cap = cv2.VideoCapture(origin_path)
        fps, count, vid_width, vid_height = self.__get_capture_attribute(cap)

        for i in range(count):
            ret, img = cap.read()
            if ret:
                img = cv2.resize(img, (vid_width, vid_height))
                cv2.imwrite(target_path + file_name + "_" + str(i) + "." + file_format,
                            self.__distortion_correction(img, vid_width, vid_height))
        cap.release()
        cv2.destroyAllWindows()

    # 获取视频参数
    @staticmethod
    def __get_capture_attribute(vid: cv2.VideoCapture):
        fps = int(vid.get(cv2.CAP_PROP_FPS))  # 视频帧率
        count = int(vid.get(cv2.CAP_PROP_FRAME_COUNT))  # 视频帧数
        width = int(vid.get(cv2.CAP_PROP_FRAME_WIDTH))  # 视频宽度 vid.shape[1]
        height = int(vid.get(cv2.CAP_PROP_FRAME_HEIGHT))  # 视频高度 vid.shape[0]
        return fps, count, width, height

    # 图片格式校对
    @staticmethod
    def __calibration_pic_format(file_format: str):
        if file_format == "jpg" or file_format == "png" or file_format == "bmp" or file_format == "tiff" or file_format == "tif":
            return file_format
        else:
            return "png"

    # 视频格式编码校对
    @staticmethod
    def __calibration_format_coding(file_format: str, file_coding: str):
        file_format, file_coding = file_format.lower(), file_coding.lower()
        if file_format == "mp4" and file_coding == "mp4v":
            return True
        elif file_format == "avi" and (file_coding == "yuv" or file_coding == "mpeg-1" or file_coding == "mpeg-4"):
            return True
        elif file_format == "ogv" and file_coding == "ogg vorbis":
            return True
        elif file_format == "flv" and file_coding == "flash":
            return True
        else:
            return False

    # 获取编码参数
    @staticmethod
    def __get_vid_coding(file_coding: str):
        file_coding = file_coding.lower()
        if file_coding == "mp4v":
            return 'mp4v'
        elif file_coding == "yuv":
            return 'I420'
        elif file_coding == "mpeg-1":
            return 'PIMI'
        elif file_coding == "mp4g-4":
            return 'XVID'
        elif file_coding == "ogg vorbis":
            return 'THEO'
        elif file_coding == "flash":
            return 'FLV1'
        else:
            return 'mp4v'

    # 畸变矫正
    def __distortion_correction(self, img, origin_width, origin_height):
        # 去畸变
        new_camera_mtx, roi = cv2.getOptimalNewCameraMatrix(self.mtx, self.dist, (origin_width, origin_height), 0,
                                                            (origin_width, origin_height))
        dst = cv2.undistort(img, self.mtx, self.dist, None, new_camera_mtx)
        # 根据ROI区域裁剪图片
        x, y, width, height = roi
        dst = dst[y:y + height, x:x + width]
        dst = cv2.resize(dst, (origin_width, origin_height))
        return dst


if __name__ == '__main__':
    cal = CameraCalibration()
    cal.calibration_init(9, 6, "./pic/*.jpg")
    cor = CameraCorrect()
    cor.correct_pic("./pic/cm_flag_5.jpg", "./output_pic/")
    cor.correct_vid2vid("./vid/vid_demo_1.mp4", "./output_vid/")
    cor.correct_vid2pic("./vid/vid_demo_1.mp4", "./output_vid/")

```