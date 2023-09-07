# Camera Calibration
## Basic Theory
* 3D좌표와 2D좌표를 사용하여 $K$ (3\*3 행렬), $R$ (3\*3 회전행렬), $t$(3\*1 이동벡터)를 찾아 내는 것
## Camera Calibration equation
* 3D좌표$( X_{\omega},  Y_{\omega},  Z_{\omega})$와 2D 이미지 투영 좌표$(u,v)$의 방정식
  -  $\begin{bmatrix} x' \\ y' \\ z' \end{bmatrix} = \mathbf{P} \begin{bmatrix} X_{\omega} \\ Y_{\omega} \\ Z_{\omega} \\1  \end{bmatrix}$
  -  $u$=$u'/_w$ $,  v$=$v'/_w$ 
  - $P$ = $K \times \begin{bmatrix} R | t \end{bmatrix}$ 
    - $K$ : Intrinsic Matrix 
    - $\begin{bmatrix} R | t \end{bmatrix}$ : Extrinsic Matrix
  - $K$ = $\begin{bmatrix} f_x & \gamma & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1\end{bmatrix}$
    - $f_x , f_y$ : 초점거리 (알반적으로 $f_x = f_y$)
    - $c_x , c_y$ : 이미지 평면에서 광학 중심의 x와y 좌표
    - $\gamma$ : 축 사이의 기울기 (일반적으로 0)

## Camera Calibration Flowchart
![camearacalibration1](/assets/img/cameracalibration1.png)

**1. 체커 보드를 통하여 3D좌표 정의**
```python
CHECKERBOARD = (11,16)
objp = np.zeros((1, CHECKERBOARD[0]*CHECKERBOARD[1], 3), np.float32)
objp[0,:,:2] = np.mgrid[0:CHECKERBOARD[0], 0:CHECKERBOARD[1]].T.reshape(-1, 2)
```
**2. 여러 시점에서 체커 보드 이미지 캡쳐**
![camearacalibration3](/assets/img/cameracalibration3.png)
**3. ChessboardCorners 메소드를 통하여 2D좌표 찾기**
```python
    flag = cv2.CALIB_CB_ADAPTIVE_THRESH+cv2.CALIB_CB_FAST_CHECK+cv2.CALIB_CB_NORMALIZE_IMAGE
    for fname in images:
        img = cv2.imread(fname)
        if _img_shape == None:
            _img_shape = img.shape[:2]
        else:
            assert _img_shape == img.shape[:2], 

        gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
        ret, corners = cv2.findChessboardCorners(gray, CHECKERBOARD, flag)
        if ret == True:
            objpoints.append(objp)
            cv2.cornerSubPix(gray,corners,(3,3),(-1,-1),subpix_criteria)
            imgpoints.append(corners)
```
**4. Camera Calibration하여 매개변수를 찾기**
```python
retval, cameraMatrix, distCoeffs, rvecs, tvecs = cv2.calibrateCamera(objectPoints, imagePoints, imageSize)
```
  - objectPoints : 3D 점 벡터로 구성된 벡터. 외부 벡터는 패턴 사진의 수만큼 요소를 포함
  - imagePoints : 2D 이미지 점 벡터로 구성된 벡터
  - imageSize : 이미지의 크기
  - cameraMatrix : 내부 카메라 행렬 ( $K$ )
  - distCoeffs : 렌즈 왜곡 계수 ( $\begin{bmatrix} R | t \end{bmatrix}$ )
  - rvecs : 회전은 3×1 벡터로 지정. 벡터의 방향은 회전 축을 지정하고 벡터의 크기는 회전 각을 지정 ( $R$ )
  - tvecs : 3×1 이동 벡터 ( $t$ )
  - 
# Experimental result
  - Origin Image

![cali6](/assets/img/cali6.jpg)

  - Find Corners

![cali5](/assets/img/cali5.jpg)

  - Result
    
![cali4](/assets/img/cali4.jpg)
