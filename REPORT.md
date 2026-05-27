# CSC4005 Lab 5 Report – Vision Transformer for Smart Campus Scene Classification

## 1. Thông tin nhóm/cá nhân
- Nhóm 9 (NLP + DL)

| STT | Họ tên | Mã sinh viên |
|---|---|---|
| 1 | Nguyễn Nam Cường | 1671040005 |
| 2 | Nguyễn Trung Thành | 1671040025 |
| 3 | Triệu Quốc Anh | 1671040002 |

- Lớp: KHMT 16-01
- Link GitHub repo: 
- Link W&B dashboard: https://wandb.ai/nguyencuong30062004-dhdn/csc4005-lab6-mit-indoor-vit?nw=nwusernguyencuong30062004

## 2. Mô tả bài toán

Trong bài lab này, bài toán cần giải quyết là phân loại ảnh không gian trong trường học bằng mô hình Vision Transformer (ViT). Mục tiêu là xác định ảnh thuộc loại phòng hoặc khu vực nào trong môi trường Smart Campus. Đây là bài toán phù hợp với bối cảnh Smart Campus vì có thể ứng dụng trong quản lý cơ sở vật chất, điều hướng thông minh, giám sát an ninh và hỗ trợ robot tự hành trong khuôn viên trường học. Dataset được lấy từ MIT Indoor Scenes 67 và sử dụng subset gồm 5 lớp: `classroom`, `computerroom`, `library`, `corridor`, và `office`. Mô hình được huấn luyện bằng transfer learning với pretrained ViT nhằm tận dụng khả năng học đặc trưng mạnh từ dữ liệu lớn.


## 3. Dữ liệu

| Nội dung | Mô tả |
|---|---|
| Dataset gốc | MIT Indoor Scenes 67 |
| Subset sử dụng | classroom, computerroom, library, corridor, office |
| Số ảnh mỗi lớp | Tổng cộng 789 ảnh |
| Train/Val/Test split | Train: 557, Validation: 116, Test: 116 |
| Tiền xử lý | resize ảnh 224×224, normalization, augmentation |


## 4. Mô hình ViT

Mô hình sử dụng kiến trúc Vision Transformer (ViT), trong đó ảnh đầu vào được chia thành các patch nhỏ, sau đó chuyển thành embedding và đưa qua Transformer Encoder để trích xuất đặc trưng trước khi phân loại bằng classification head.

```text
image → patch embedding → positional embedding → transformer encoder → classification head

Thông số: Cấu hình best model (ViT head-only)

| Thành phần       | Giá trị    |
| ---------------- | ---------- |
| model_name       | vit_b_16   |
| train_mode       | head_only  |
| img_size         | 224        |
| batch size       | 8          |
| số epoch         | 10         |
| learning rate    | 0.001      |
| optimizer        | AdamW      |
| total params     | 85,802,501 |
| trainable params | 3,845      |
| trainable ratio  | 0.0000448  |

```

## 5. Kết quả

| Metric     | Validation |   Test |
| ---------- | ---------: | -----: |
| Accuracy   |     0.9397 | 0.9138 |
| Macro-F1   |     0.9321 | 0.8811 |
| Best epoch |         10 |     10 |

Nhận xét

Mô hình đạt kết quả khá tốt trên cả validation set và test set. Validation Macro-F1 đạt khoảng 93.2%, cho thấy mô hình có khả năng phân loại tương đối đồng đều giữa các lớp. Test accuracy đạt hơn 91%, chứng tỏ mô hình có khả năng tổng quát hóa tốt dù chỉ fine-tune classification head.

Chèn ảnh:

- Learning curves

<img width="400" height="350" alt="Lab5-baseline-confusion-matrix" src="https://github.com/user-attachments/assets/6701f743-64cf-4c7e-9249-2955999946e7" />

- Confusion matrix

<img width="800" height="444" alt="Lab5-baseline-curves" src="https://github.com/user-attachments/assets/545444b5-cbc3-43d3-a044-572dfac7084a" />


## 6. Phân tích lỗi

- Lớp mô hình dự đoán tốt nhất là `corridor` vì có số lượng dự đoán đúng rất cao và rất ít mẫu bị nhầm lẫn.

- Lớp dễ bị nhầm nhất là `computerroom`.

- Cặp lớp dễ nhầm với nhau nhất là `computerroom` và office vì cả hai đều chứa bàn ghế, máy tính và bố cục không gian tương đối giống nhau. Ngoài ra `classroom` đôi lúc cũng bị nhầm với `computerroom`.

- Dataset không bị mất cân bằng quá nghiêm trọng, tuy nhiên số lượng ảnh giữa các lớp vẫn có chênh lệch nhẹ.

- Augmentation giúp cải thiện khả năng tổng quát hóa của mô hình bằng cách tạo ra nhiều biến thể dữ liệu hơn, từ đó giảm overfitting trên tập train.

## 7. Liên hệ với lý thuyết ViT

- Patch embedding trong ViT tương tự bước word embedding trong NLP, khi các patch ảnh được chuyển thành vector embedding trước khi đưa vào Transformer.
- ViT cần positional embedding vì Transformer không tự hiểu được vị trí không gian của các patch trong ảnh.

- `head_only` train nhanh hơn finetune vì chỉ cập nhật trọng số của classification head thay vì toàn bộ backbone ViT.

- Nên fine-tune toàn bộ backbone khi dataset đủ lớn hoặc khi domain dữ liệu khác biệt nhiều so với dữ liệu pretrained ban đầu.

## 8. W&B evidence

- Link run: https://wandb.ai/nguyencuong30062004-dhdn/csc4005-lab6-mit-indoor-vit?nw=nwusernguyencuong30062004

- Các hyperparameter chính: 
    
    + learning rate = 0.001

    + batch size = 8

    + epochs = 10

    + optimizer = AdamW

- Các metric được log:
    
    + train_loss
    
    + val_loss
    
    + train_accuracy
    
    + val_accuracy
    
    + macro_f1
    
    + learning rate

## 9. Kết luận

Trong bài lab này, mô hình Vision Transformer đã đạt kết quả tốt trên bài toán phân loại cảnh trong môi trường Smart Campus. Cấu hình pretrained ViT head-only cho khả năng tổng quát hóa ổn định với validation Macro-F1 khoảng 93% và test accuracy hơn 91%. ViT có ưu điểm là học được đặc trưng mạnh nhờ pretrained model và hoạt động hiệu quả ngay cả khi chỉ fine-tune classification head. Tuy nhiên, với dataset nhỏ, fine-tune toàn bộ backbone dễ dẫn đến overfitting và tốn nhiều tài nguyên tính toán hơn. Nếu tiếp tục cải thiện mô hình, có thể tập trung vào augmentation mạnh hơn, mở rộng dataset hoặc áp dụng regularization và early stopping hiệu quả hơn.
