# DinoRunner - Troubleshooting Guide

## 🔥 Vấn đề Firebase Authentication & Database

### Vấn đề hiện tại:
- ✅ Có thể đăng ký và đăng nhập
- ❌ Thông tin user không xuất hiện trong Firestore database
- ❌ Không thể đăng nhập với tài khoản có sẵn

## 🔍 Các bước kiểm tra và sửa lỗi

### Bước 1: Kiểm tra Firebase Setup

1. **Kiểm tra file cấu hình:**
   ```
   android/app/google-services.json
   ```
   - Đảm bảo file này tồn tại
   - Package name phải là: `com.example.endlessrunner`

2. **Kiểm tra Firebase Console:**
   - Vào [Firebase Console](https://console.firebase.google.com/)
   - Chọn project của bạn
   - Kiểm tra Authentication tab có users không

### Bước 2: Kiểm tra Firestore Security Rules

⚠️ **QUAN TRỌNG**: Đây có thể là nguyên nhân chính!

1. **Vào Firebase Console → Firestore Database → Rules**

2. **Thay thế rules hiện tại bằng:**

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

3. **Nhấn "Publish" để áp dụng rules mới**

### Bước 3: Sử dụng Debug Tool

1. **Chạy app và đăng nhập**
2. **Trong Main Menu, nhấn nút "Debug Firebase"** (chỉ hiện trong debug mode)
3. **Kiểm tra console output để xem lỗi cụ thể**

### Bước 4: Test theo steps

#### Test 1: Authentication
```
flutter run
→ Đăng nhập
→ Kiểm tra có navigate đến main menu không
→ Nhấn "Debug Firebase" 
→ Xem console output
```

#### Test 2: Database Write
1. Đăng ký tài khoản mới
2. Kiểm tra Firebase Console → Firestore Database
3. Tìm collections: `users`, `players`, `settings`
4. Xem có documents với UID của user không

#### Test 3: Permissions
```
→ Login vào app
→ Nhấn "Debug Firebase"
→ Xem phần "Testing Firestore Permissions"
→ Nếu có lỗi permission denied → Security Rules chưa đúng
```

### Bước 5: Các giải pháp cụ thể

#### Nếu "Permission denied":
1. Kiểm tra Security Rules (Bước 2)
2. Đảm bảo user đã đăng nhập (check Authentication tab)
3. UID trong rules phải match với UID thực tế

#### Nếu "Document not found":
1. Kiểm tra RegisterScreen có gọi `saveToFirestore()` không
2. Kiểm tra collection names đúng chưa
3. Thử đăng ký tài khoản mới

#### Nếu "Connection failed":
1. Kiểm tra internet connection
2. Kiểm tra `google-services.json` file
3. Restart app sau khi thay đổi config

### Bước 6: Clean & Rebuild

```bash
flutter clean
flutter pub get
flutter run
```

### Bước 7: Test với tài khoản mới

1. **Đăng ký tài khoản hoàn toàn mới:**
   - Email: test@test.com
   - Password: Test123!
   - Display Name: TestUser

2. **Kiểm tra ngay trong Firebase Console:**
   - Authentication → Users (phải có user mới)
   - Firestore Database → Data → users (phải có document)
   - Firestore Database → Data → players (phải có document)
   - Firestore Database → Data → settings (phải có document)

## 🚨 Các lỗi thường gặp

### Lỗi 1: "PERMISSION_DENIED"
**Nguyên nhân:** Security Rules chưa được cập nhật đúng
**Giải pháp:** Thực hiện Bước 2 ở trên

### Lỗi 2: "Invalid email or password"
**Nguyên nhân:** 
- Password không đúng format (cần uppercase + special char + 6+ chars)
- Email chưa được verify
**Giải pháp:** 
- Kiểm tra email inbox để verify
- Đăng ký lại với password strong

### Lỗi 3: "User already exists"
**Nguyên nhân:** Email đã được đăng ký
**Giải pháp:** Sử dụng email khác hoặc reset password

### Lỗi 4: Documents không được tạo
**Nguyên nhân:** 
- Security Rules chặn write operations
- Code không gọi saveToFirestore()
**Giải pháp:**
- Update Security Rules
- Kiểm tra RegisterScreen code

## 📱 Debug Commands

### Kiểm tra logs chi tiết:
```bash
flutter logs
```

### Chạy với verbose output:
```bash
flutter run -v
```

### Check Firebase connection:
```bash
flutter doctor
```

## 🔧 Recovery Steps

Nếu vẫn không hoạt động:

1. **Backup current code**
2. **Xóa và tạo lại Firebase project**
3. **Tạo mới google-services.json**
4. **Apply Security Rules từ đầu**
5. **Test với tài khoản mới**

## 📞 Support

Nếu vấn đề vẫn tiếp tục:
1. Chụp screenshot console output từ "Debug Firebase"
2. Chụp screenshot Firebase Console (Authentication + Firestore)
3. Cung cấp thông tin để được support cụ thể 