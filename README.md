# Phần Thành viên 3 - Bộ test, đánh giá định lượng, phân tích thất bại

Đề tài: [CVPR'25] HVI: A New Color Space for Low-light Image Enhancement.
Sản phẩm theo bảng phân công: `test_cases/`, `quantitative_results.csv`, bộ ảnh 4 ca thất bại,
cùng hai script `eval.py` và `measure.py`.

Trước khi chạy file chay_tv3.ipynb thì hãy tải file của bải báo rồi đưa vào trong này.

**Trạng thái:** script đã chạy thử trên dữ liệu giả. Chưa có ảnh thật, chưa có kết quả chạy HVI-CIDNet,
vì cần tải bộ dữ liệu, trọng số và chạy mô hình trên máy của bạn (hoặc Colab). Mọi dòng trong
`quantitative_results.csv` có `source = bai_bao_*` là số trích từ bài báo; dòng `nhom_tu_chay`
đang để trống cho tới khi bạn chạy `eval.py evaluate`.

## Cài đặt

```
pip install numpy opencv-python scikit-image pandas matplotlib
pip install torch lpips pyiqa        # cho LPIPS, NIQE, BRISQUE; thiếu thì các chỉ số này là NaN
```

## Các bước

1. **Lấy mã và trọng số HVI-CIDNet** từ https://github.com/Fediory/HVI-CIDNet; làm theo README của repo
   để cài môi trường và tải trọng số (LOL-v1, LOL-v2) và bộ dữ liệu.
2. **Xếp ảnh test** theo cấu trúc (tên file ở `input/` và `gt/` phải trùng nhau):
   ```
   test_cases/LOLv1/{input,gt}        15 ảnh
   test_cases/LOLv2_Real/{input,gt}   100 ảnh
   test_cases/DICM/input              69 ảnh
   test_cases/LIME/input              10 ảnh
   test_cases/custom_night/input      20 ảnh tự chụp
   ```
   Kiểm tra: `python prepare_test_cases.py check`
3. **Chụp 20 ảnh** (xem mục "Hướng dẫn chụp" dưới đây), đặt trong một thư mục, tên `red_01.jpg`,
   `night_01.jpg`, `backlit_01.jpg`, `dark_01.jpg`..., rồi:
   `python prepare_test_cases.py custom --raw <thư_mục_ảnh_gốc>`
4. **Thống kê lược đồ xám:** `python prepare_test_cases.py hist` (số liệu để thay cho nhận định "85%").
5. **Chạy HVI-CIDNet** bằng mã chính thức trên từng `test_cases/<DATASET>/input`, chép ảnh ra
   `results/<DATASET>/HVI-CIDNet/`. Trong notebook, Bước 7 làm việc này bằng chế độ `eval.py --unpaired --custome --custome_path ...`
   của repo (xem README repo, mục Testing). Repo cũng có `eval.py`, `measure.py` trùng tên: chạy chúng trong thư mục `HVI-CIDNet/`. Bản tự code của TV1 tạo ở bước 6 bằng `eval.py scratch`.
6. **Tạo ảnh đối chứng:** `python eval.py baselines` (OpenCV) và `python eval.py scratch` (tự code TV1); cùng tham số CLAHE clip 2.0, lưới 8x8, Gamma 0.4
7. **Đo và ghi CSV:** `python eval.py evaluate`
   Sinh `quantitative_results.csv` (dòng bài báo + dòng nhóm tự chạy) và `per_image_results.csv`.
   Có thể đo riêng một thư mục: `python measure.py --pred results/LOLv1/HVI-CIDNet --gt test_cases/LOLv1/gt --gt-mean`
8. **Ca thất bại:** xem mục dưới.

Quy ước đo: PSNR/SSIM tính trên ảnh RGB 8 bit (SSIM Gaussian 11x11, sigma 1.5); LPIPS dùng AlexNet như bài báo;
NIQE/BRISQUE qua `pyiqa`. GT Mean chỉ bật cho LOLv1, đúng như bài báo (Mục 10.2, vì chỉ có 15 ảnh test).
Số của bạn có thể lệch nhẹ so với bài báo vì cài đặt SSIM/LPIPS khác nhau; hãy ghi rõ cách đo trong báo cáo.

Lưu ý công thức GT Mean: Eq. 21 trong bài báo ghi q = mean(out) / mean(gt) rồi I_q = q · I_out. Viết như vậy thì
độ sáng trung bình sau chỉnh không khớp ảnh chuẩn (đã thử: ảnh ra 162,4, ảnh chuẩn 119,7, sau Eq. 21 literal ra 192,1).
`measure.py` dùng q = mean(gt) / mean(out), là chiều khớp với mục đích nêu trong bài báo.

## Hướng dẫn chụp 20 ảnh (bảng phân công: 20 ảnh thực tế ban đêm)

Gợi ý phân bổ: 6 `red` (vật thể màu đỏ trong tối), 6 `night` (cảnh đêm), 4 `backlit` (có đèn pha/đèn đường
trong khung), 4 `dark` (tối kín, gần như đen). Chụp ngang, cùng một điện thoại, tắt chế độ ban đêm/HDR/AI của máy
(nếu không ảnh gốc đã bị tăng sáng), giữ ISO và phơi sáng cố định hoặc ghi lại thông số. Nếu có chân máy, chụp thêm
một ảnh đủ sáng của cùng cảnh để làm ảnh chuẩn (khi đó mới đo được PSNR/SSIM cho ảnh tự chụp).

## Ca thất bại

Thư mục: `test_cases/failure_cases/{case1_color_shift, case2_k_alpha, case3_total_dark, case4_backlit}/input`
(thêm `gt/` nếu có), ảnh đầu ra ở `results/failure_cases/<ca>/<METHOD>/` cùng tên file.

```
python failure_cases.py k_sweep --image test_cases/LOLv1/input/22.png     # tái hiện Hình 9, không cần mô hình
python failure_cases.py panel   --case case4_backlit --methods opencv_clahe HVI-CIDNet
python failure_cases.py metrics --case case4_backlit --methods opencv_clahe HVI-CIDNet
```

`metrics` ghi `failure_case_metrics.csv`: `sat_pct` (cháy sáng, ca 4), `dark_detail` và `noise_dark` (vùng rất tối,
ca 2 và 3), `rg_ratio`/`bg_ratio` (lệch màu, ca 1). Phần nội dung và điểm cần sửa trong khung báo cáo nằm trong tài liệu
"Phần Thành viên 3" đi kèm.

Ghi chú về k: bài báo huấn luyện k như một tham số học được và nói có thể huấn luyện mạng riêng cho từng k. Đổi k lúc
suy luận trên trọng số đã huấn luyện không tương đương thí nghiệm của bài báo, nên `k_sweep` chỉ trực quan hóa bản đồ
HV (như Hình 8, 9), không chạy lại mạng.
