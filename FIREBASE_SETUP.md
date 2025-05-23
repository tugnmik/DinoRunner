# Firebase Setup Guide for DinoRunner Multiplayer

## Thiết lập Firebase

### 1. Tạo dự án Firebase

1. Truy cập [Firebase Console](https://console.firebase.google.com/)
2. Nhấn "Add project" hoặc "Create a project"
3. Đặt tên cho dự án (ví dụ: "DinoRunner")
4. Tắt Google Analytics nếu không cần thiết
5. Nhấn "Create project"

### 2. Thêm ứng dụng Android

1. Trong Firebase Console, nhấn biểu tượng Android
2. Nhập package name: `com.example.endlessrunner`
3. Nhập nickname cho app (ví dụ: "DinoRunner Android")
4. Tải file `google-services.json`
5. Đặt file vào thư mục `android/app/`

### 3. Thêm ứng dụng iOS (nếu cần)

1. Nhấn biểu tượng iOS
2. Nhập bundle ID: `com.example.endlessrunner`
3. Tải file `GoogleService-Info.plist`
4. Đặt file vào thư mục `ios/Runner/`

### 4. Thêm ứng dụng Web (nếu cần)

1. Nhấn biểu tượng Web
2. Nhập nickname cho web app
3. Copy Firebase config và đặt vào `web/index.html`

### 5. Thiết lập Firestore Database

1. Trong Firebase Console, vào "Firestore Database"
2. Nhấn "Create database"
3. Chọn "Start in test mode" (cho development)
4. Chọn location gần với người dùng

### 6. Cấu hình Authentication

1. Vào "Authentication" → "Sign-in method"
2. Bật "Email/Password" provider
3. Có thể bật thêm Google, Facebook, v.v. nếu cần

### 7. Thiết lập Security Rules cho Firestore

⚠️ **QUAN TRỌNG**: Thay thế rules mặc định bằng rules sau để đảm bảo tương thích với cả hệ thống cũ và multiplayer mới:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Rules for existing users collection
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Rules for existing players collection
    match /players/{playerId} {
      allow read, write: if request.auth != null && request.auth.uid == playerId;
    }
    
    // Rules for existing settings collection
    match /settings/{settingId} {
      allow read, write: if request.auth != null && request.auth.uid == settingId;
    }
    
    // Rules for NEW multiplayer rooms collection
    match /rooms/{roomId} {
      allow read, write: if request.auth != null;
    }
    
    // Rules for NEW multiplayer game_sessions collection  
    match /game_sessions/{sessionId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

### 8. Cấu trúc dữ liệu Firestore

#### Existing Collections (từ code cũ):

##### Collection: `users`
```
users/{userId}
├── uid: string
├── email: string
├── password: string (encrypted)
└── display_name: string
```

##### Collection: `players`
```
players/{playerId}
├── uid: string
├── lives: number
├── health: number
├── current_score: number
├── highScore: number
└── datetime: timestamp
```

##### Collection: `settings`
```
settings/{settingId}
├── uid: string
├── bgmVolume: number
├── sfxVolume: number
└── theme: string
```

#### NEW Multiplayer Collections:

##### Collection: `rooms`
```
rooms/{roomId}
├── roomId: string (4 digits)
├── player1: {
│   ├── userId: string
│   ├── username: string
│   ├── isReady: boolean
│   └── isHost: boolean
├── player2: {
│   ├── userId: string
│   ├── username: string
│   ├── isReady: boolean
│   └── isHost: boolean
├── gameState: string ("waiting", "playing", "finished")
└── createdAt: timestamp
```

##### Collection: `game_sessions`
```
game_sessions/{roomId}
├── roomId: string
├── player1Data: {
│   ├── userId: string
│   ├── lives: number
│   ├── health: number
│   └── status: string ("alive", "dead")
├── player2Data: {
│   ├── userId: string
│   ├── lives: number
│   ├── health: number
│   └── status: string ("alive", "dead")
└── lastUpdated: timestamp
```

### 9. Chạy ứng dụng

1. Đảm bảo đã đặt `google-services.json` vào đúng vị trí
2. Chạy lệnh: `flutter pub get`
3. Chạy ứng dụng: `flutter run`

## Tính năng Multiplayer

### Luồng hoạt động:

1. **Tạo phòng**: Người chơi tạo phòng với Room ID 4 chữ số ngẫu nhiên
2. **Tham gia phòng**: Người chơi khác nhập Room ID để tham gia
3. **Sẵn sàng**: Cả hai người chơi nhấn nút "Ready"
4. **Bắt đầu game**: Game tự động bắt đầu khi cả hai sẵn sàng
5. **Gameplay**: 
   - Hiển thị lives và health của cả hai người chơi
   - Không có score và pause button
   - Cập nhật trạng thái lên Firebase mỗi 500ms
6. **Kết thúc**: Người chơi nào còn sống khi đối thủ chết sẽ thắng

### Lưu ý:
- Game không tự động pause khi chuyển tab/app để duy trì tính cạnh tranh real-time
- Dữ liệu game session sẽ được xóa khi game kết thúc
- Room sẽ được dọn dẹp khi người chơi rời khỏi

## Troubleshooting

### Lỗi thường gặp:

1. **"FirebaseApp not initialized"**
   - Đảm bảo `google-services.json` đã được đặt đúng vị trí
   - Kiểm tra package name trong file config

2. **"Permission denied"**
   - ⚠️ **KIỂM TRA SECURITY RULES**: Đảm bảo đã cập nhật rules như trên
   - Đảm bảo người dùng đã đăng nhập
   - Kiểm tra collection names đúng

3. **"User data not showing in database"**
   - Kiểm tra Security Rules đã bao gồm collection `users`
   - Xác nhận user đã đăng ký thành công
   - Kiểm tra Firestore Console để xem data có được tạo không

4. **"Can't login with existing account"**
   - Xóa cache app và thử lại
   - Kiểm tra email verification
   - Đảm bảo password đúng format (uppercase, special char, 6+ chars)

5. **"Room not found"**
   - Kiểm tra Room ID có đúng 4 chữ số
   - Đảm bảo phòng tồn tại và chưa bị xóa

### Debug mode:
- Mở Firestore Console để theo dõi dữ liệu real-time
- Kiểm tra Authentication tab để xem users đã registered
- Sử dụng Flutter DevTools để debug

### Test Steps để đảm bảo everything works:

1. **Test Authentication:**
   - Đăng ký tài khoản mới → kiểm tra `users` collection có data
   - Đăng nhập → kiểm tra có navigate đến main menu không
   - Kiểm tra `players` và `settings` collections được tạo

2. **Test Multiplayer:**
   - Tạo room → kiểm tra `rooms` collection
   - Join room → kiểm tra player2 được thêm vào
   - Start game → kiểm tra `game_sessions` collection được tạo
   - Gameplay → kiểm tra data updates real-time 