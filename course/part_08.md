# Part 08: Forms และ Input Handling
## ขั้นตอนที่ 71-80

---

## สารบัญ
1. [TextField พื้นฐาน](#ขั้นตอนที่-71-textfield-พื้นฐาน)
2. [TextFormField & Form](#ขั้นตอนที่-72-textformfield--form)
3. [Validators](#ขั้นตอนที่-73-validators)
4. [Checkbox, Radio, Switch](#ขั้นตอนที่-74-checkbox-radio-switch)
5. [Slider & RangeSlider](#ขั้นตอนที่-75-slider--rangeslider)
6. [DropdownButton & DropdownMenu](#ขั้นตอนที่-76-dropdownbutton--dropdownmenu)
7. [DatePicker & TimePicker](#ขั้นตอนที่-77-datepicker--timepicker)
8. [Autocomplete & SearchAnchor](#ขั้นตอนที่-78-autocomplete--searchanchor)
9. [FocusNode & TextEditingController](#ขั้นตอนที่-79-focusnode--texteditingcontroller)
10. [Workshop: Registration Form](#ขั้นตอนที่-80-workshop-registration-form)

---

## ขั้นตอนที่ 71: TextField พื้นฐาน

TextField เป็น widget สำหรับรับข้อความจากผู้ใช้

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(const FormsApp());

class FormsApp extends StatelessWidget {
  const FormsApp({super.key});
  @override
  Widget build(BuildContext context) => MaterialApp(
        theme: ThemeData(useMaterial3: true),
        home: const TextFieldDemo(),
      );
}

class TextFieldDemo extends StatefulWidget {
  const TextFieldDemo({super.key});

  @override
  State<TextFieldDemo> createState() => _TextFieldDemoState();
}

class _TextFieldDemoState extends State<TextFieldDemo> {
  String _value = '';
  bool _obscureText = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('TextField')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. TextField พื้นฐาน
            const Text('1. พื้นฐาน:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            TextField(
              onChanged: (value) => setState(() => _value = value),
              decoration: const InputDecoration(
                labelText: 'ชื่อ',
                hintText: 'กรอกชื่อของคุณ',
                border: OutlineInputBorder(),
              ),
            ),
            Text('ค่าที่กรอก: $_value', style: const TextStyle(color: Colors.grey)),

            const SizedBox(height: 16),

            // 2. Password Field
            const Text('2. Password:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            TextField(
              obscureText: _obscureText,
              decoration: InputDecoration(
                labelText: 'รหัสผ่าน',
                prefixIcon: const Icon(Icons.lock_outline),
                suffixIcon: IconButton(
                  icon: Icon(
                    _obscureText ? Icons.visibility : Icons.visibility_off,
                  ),
                  onPressed: () =>
                      setState(() => _obscureText = !_obscureText),
                ),
                border: const OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // 3. TextField ที่มี prefix/suffix
            const Text('3. Prefix & Suffix:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            TextField(
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(
                labelText: 'ราคา',
                prefixText: '฿ ',
                suffixText: '.00',
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 8),

            TextField(
              decoration: const InputDecoration(
                labelText: 'ค้นหา',
                prefixIcon: Icon(Icons.search),
                suffixIcon: Icon(Icons.clear),
                filled: true,
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.all(Radius.circular(30)),
                  borderSide: BorderSide.none,
                ),
              ),
            ),

            const SizedBox(height: 16),

            // 4. Multiline TextField
            const Text('4. Multiline:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const TextField(
              maxLines: 4,
              minLines: 3,
              decoration: InputDecoration(
                labelText: 'หมายเหตุ',
                hintText: 'กรอกหมายเหตุ...',
                border: OutlineInputBorder(),
                alignLabelWithHint: true,
              ),
            ),

            const SizedBox(height: 16),

            // 5. InputFormatters - จำกัดรูปแบบ input
            const Text('5. InputFormatters:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            TextField(
              keyboardType: TextInputType.number,
              inputFormatters: [
                FilteringTextInputFormatter.digitsOnly,  // ตัวเลขเท่านั้น
                LengthLimitingTextInputFormatter(10),     // จำกัด 10 ตัว
              ],
              decoration: const InputDecoration(
                labelText: 'เบอร์โทร (ตัวเลขเท่านั้น)',
                hintText: '0812345678',
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 8),

            TextField(
              textCapitalization: TextCapitalization.characters,  // uppercase
              inputFormatters: [
                FilteringTextInputFormatter.allow(RegExp(r'[A-Za-z]')),
              ],
              decoration: const InputDecoration(
                labelText: 'รหัสประจำตัว (ตัวอักษรเท่านั้น)',
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // 6. Keyboard Types
            const Text('6. Keyboard Types:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            const TextField(
              keyboardType: TextInputType.emailAddress,
              decoration: InputDecoration(
                labelText: 'อีเมล',
                prefixIcon: Icon(Icons.email_outlined),
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 8),

            const TextField(
              keyboardType: TextInputType.phone,
              decoration: InputDecoration(
                labelText: 'เบอร์โทร',
                prefixIcon: Icon(Icons.phone_outlined),
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 8),

            const TextField(
              keyboardType: TextInputType.url,
              decoration: InputDecoration(
                labelText: 'เว็บไซต์',
                prefixIcon: Icon(Icons.language),
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // 7. TextInputAction
            const Text('7. TextInputAction:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            const TextField(
              textInputAction: TextInputAction.next,
              decoration: InputDecoration(
                labelText: 'Field 1 (next)',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 8),
            const TextField(
              textInputAction: TextInputAction.done,
              decoration: InputDecoration(
                labelText: 'Field 2 (done)',
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // 8. Counter Text
            const Text('8. Counter:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const TextField(
              maxLength: 100,
              maxLines: 3,
              decoration: InputDecoration(
                labelText: 'ชีวประวัติ',
                hintText: 'บอกเล่าเกี่ยวกับตัวเอง...',
                border: OutlineInputBorder(),
                alignLabelWithHint: true,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 72: TextFormField & Form

Form widget รวม TextFormField หลายตัวและจัดการ validation

```dart
import 'package:flutter/material.dart';

class FormDemoScreen extends StatefulWidget {
  const FormDemoScreen({super.key});

  @override
  State<FormDemoScreen> createState() => _FormDemoScreenState();
}

class _FormDemoScreenState extends State<FormDemoScreen> {
  // GlobalKey สำหรับ control Form
  final _formKey = GlobalKey<FormState>();

  // Controllers สำหรับอ่านค่า
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  bool _autoValidate = false;
  bool _obscurePassword = true;

  @override
  void dispose() {
    // ต้อง dispose controllers!
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('TextFormField & Form')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          autovalidateMode: _autoValidate
              ? AutovalidateMode.onUserInteraction
              : AutovalidateMode.disabled,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // Name
              TextFormField(
                controller: _nameController,
                textCapitalization: TextCapitalization.words,
                textInputAction: TextInputAction.next,
                decoration: const InputDecoration(
                  labelText: 'ชื่อ-นามสกุล *',
                  prefixIcon: Icon(Icons.person_outline),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.trim().isEmpty) {
                    return 'กรุณากรอกชื่อ-นามสกุล';
                  }
                  if (value.trim().length < 2) {
                    return 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร';
                  }
                  return null; // valid
                },
              ),

              const SizedBox(height: 16),

              // Email
              TextFormField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                textInputAction: TextInputAction.next,
                decoration: const InputDecoration(
                  labelText: 'อีเมล *',
                  prefixIcon: Icon(Icons.email_outlined),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'กรุณากรอกอีเมล';
                  }
                  final emailRegex =
                      RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
                  if (!emailRegex.hasMatch(value)) {
                    return 'รูปแบบอีเมลไม่ถูกต้อง';
                  }
                  return null;
                },
              ),

              const SizedBox(height: 16),

              // Password
              TextFormField(
                controller: _passwordController,
                obscureText: _obscurePassword,
                textInputAction: TextInputAction.next,
                decoration: InputDecoration(
                  labelText: 'รหัสผ่าน *',
                  prefixIcon: const Icon(Icons.lock_outline),
                  suffixIcon: IconButton(
                    icon: Icon(
                      _obscurePassword
                          ? Icons.visibility
                          : Icons.visibility_off,
                    ),
                    onPressed: () => setState(
                        () => _obscurePassword = !_obscurePassword),
                  ),
                  border: const OutlineInputBorder(),
                  helperText: 'อย่างน้อย 8 ตัว มีตัวเลขและตัวอักษร',
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'กรุณากรอกรหัสผ่าน';
                  }
                  if (value.length < 8) {
                    return 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร';
                  }
                  if (!value.contains(RegExp(r'[0-9]'))) {
                    return 'ต้องมีตัวเลขอย่างน้อย 1 ตัว';
                  }
                  if (!value.contains(RegExp(r'[A-Za-z]'))) {
                    return 'ต้องมีตัวอักษรอย่างน้อย 1 ตัว';
                  }
                  return null;
                },
              ),

              const SizedBox(height: 16),

              // Confirm Password
              TextFormField(
                obscureText: _obscurePassword,
                textInputAction: TextInputAction.done,
                decoration: const InputDecoration(
                  labelText: 'ยืนยันรหัสผ่าน *',
                  prefixIcon: Icon(Icons.lock_outline),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value != _passwordController.text) {
                    return 'รหัสผ่านไม่ตรงกัน';
                  }
                  return null;
                },
              ),

              const SizedBox(height: 24),

              // Auto-validate toggle
              SwitchListTile(
                title: const Text('Validate อัตโนมัติ'),
                subtitle: const Text('แสดง error ขณะพิมพ์'),
                value: _autoValidate,
                onChanged: (v) => setState(() => _autoValidate = v),
              ),

              const SizedBox(height: 16),

              // Submit Button
              ElevatedButton(
                onPressed: _handleSubmit,
                child: const Text('ส่งข้อมูล'),
              ),

              const SizedBox(height: 8),

              // Reset Button
              OutlinedButton(
                onPressed: () {
                  _formKey.currentState?.reset();
                  _nameController.clear();
                  _emailController.clear();
                  _passwordController.clear();
                },
                child: const Text('รีเซ็ต'),
              ),
            ],
          ),
        ),
      ),
    );
  }

  void _handleSubmit() {
    // Validate ทั้ง Form
    if (_formKey.currentState?.validate() == true) {
      // Save ค่าทั้งหมด
      _formKey.currentState?.save();

      // อ่านค่าจาก controllers
      final name = _nameController.text.trim();
      final email = _emailController.text.trim();

      // แสดงผลลัพธ์
      showDialog(
        context: context,
        builder: (context) => AlertDialog(
          title: const Text('สมัครสมาชิกสำเร็จ! 🎉'),
          content: Text('ชื่อ: $name\nอีเมล: $email'),
          actions: [
            ElevatedButton(
              onPressed: () => Navigator.pop(context),
              child: const Text('ตกลง'),
            ),
          ],
        ),
      );
    } else {
      // ถ้า invalid - auto validate
      setState(() => _autoValidate = true);
    }
  }
}
```

---

## ขั้นตอนที่ 73: Validators

Validators ตรวจสอบความถูกต้องของข้อมูล

```dart
import 'package:flutter/material.dart';

// Utility class สำหรับ validators
class Validators {
  // Required
  static String? required(String? value, {String? message}) {
    if (value == null || value.trim().isEmpty) {
      return message ?? 'กรุณากรอกข้อมูล';
    }
    return null;
  }

  // Email
  static String? email(String? value) {
    if (value == null || value.isEmpty) return 'กรุณากรอกอีเมล';
    final regex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
    if (!regex.hasMatch(value)) return 'รูปแบบอีเมลไม่ถูกต้อง';
    return null;
  }

  // Thai Phone Number
  static String? thaiPhone(String? value) {
    if (value == null || value.isEmpty) return 'กรุณากรอกเบอร์โทร';
    final cleaned = value.replaceAll(RegExp(r'[\s\-\(\)]'), '');
    if (!RegExp(r'^(0[0-9]{8,9})$').hasMatch(cleaned)) {
      return 'รูปแบบเบอร์โทรไม่ถูกต้อง (เช่น 0812345678)';
    }
    return null;
  }

  // Thai National ID
  static String? nationalId(String? value) {
    if (value == null || value.isEmpty) return 'กรุณากรอกเลขบัตรประชาชน';
    final cleaned = value.replaceAll('-', '');
    if (cleaned.length != 13 || !RegExp(r'^[0-9]+$').hasMatch(cleaned)) {
      return 'เลขบัตรประชาชนต้องมี 13 หลัก';
    }
    // Checksum validation
    int sum = 0;
    for (int i = 0; i < 12; i++) {
      sum += int.parse(cleaned[i]) * (13 - i);
    }
    final checkDigit = (11 - (sum % 11)) % 10;
    if (checkDigit != int.parse(cleaned[12])) {
      return 'เลขบัตรประชาชนไม่ถูกต้อง';
    }
    return null;
  }

  // Password Strength
  static String? password(String? value, {int minLength = 8}) {
    if (value == null || value.isEmpty) return 'กรุณากรอกรหัสผ่าน';
    if (value.length < minLength) {
      return 'รหัสผ่านต้องมีอย่างน้อย $minLength ตัวอักษร';
    }
    if (!value.contains(RegExp(r'[A-Z]'))) {
      return 'ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว';
    }
    if (!value.contains(RegExp(r'[a-z]'))) {
      return 'ต้องมีตัวพิมพ์เล็กอย่างน้อย 1 ตัว';
    }
    if (!value.contains(RegExp(r'[0-9]'))) {
      return 'ต้องมีตัวเลขอย่างน้อย 1 ตัว';
    }
    if (!value.contains(RegExp(r'[!@#\$&*~]'))) {
      return 'ต้องมีอักขระพิเศษอย่างน้อย 1 ตัว (!@#\$&*~)';
    }
    return null;
  }

  // URL
  static String? url(String? value) {
    if (value == null || value.isEmpty) return null; // optional
    final regex = RegExp(
        r'^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$');
    if (!regex.hasMatch(value)) return 'URL ไม่ถูกต้อง';
    return null;
  }

  // Combine validators
  static String? Function(String?) compose(
      List<String? Function(String?)> validators) {
    return (value) {
      for (final validator in validators) {
        final error = validator(value);
        if (error != null) return error;
      }
      return null;
    };
  }

  // Min Length
  static String? Function(String?) minLength(int min, {String? message}) {
    return (value) {
      if (value != null && value.length < min) {
        return message ?? 'ต้องมีอย่างน้อย $min ตัวอักษร';
      }
      return null;
    };
  }

  // Max Length
  static String? Function(String?) maxLength(int max, {String? message}) {
    return (value) {
      if (value != null && value.length > max) {
        return message ?? 'ต้องไม่เกิน $max ตัวอักษร';
      }
      return null;
    };
  }
}

// Password Strength Indicator
class PasswordStrengthIndicator extends StatelessWidget {
  final String password;

  const PasswordStrengthIndicator({super.key, required this.password});

  int get _strength {
    int score = 0;
    if (password.length >= 8) score++;
    if (password.contains(RegExp(r'[A-Z]'))) score++;
    if (password.contains(RegExp(r'[a-z]'))) score++;
    if (password.contains(RegExp(r'[0-9]'))) score++;
    if (password.contains(RegExp(r'[!@#\$&*~]'))) score++;
    return score;
  }

  String get _label {
    switch (_strength) {
      case 0:
      case 1: return 'อ่อนมาก';
      case 2: return 'อ่อน';
      case 3: return 'ปานกลาง';
      case 4: return 'แข็งแกร่ง';
      case 5: return 'แข็งแกร่งมาก';
      default: return '';
    }
  }

  Color get _color {
    switch (_strength) {
      case 0:
      case 1: return Colors.red;
      case 2: return Colors.orange;
      case 3: return Colors.yellow.shade700;
      case 4: return Colors.lightGreen;
      case 5: return Colors.green;
      default: return Colors.grey;
    }
  }

  @override
  Widget build(BuildContext context) {
    if (password.isEmpty) return const SizedBox.shrink();

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Row(
          children: List.generate(5, (index) {
            return Expanded(
              child: Container(
                margin: const EdgeInsets.only(right: 4),
                height: 4,
                decoration: BoxDecoration(
                  color: index < _strength ? _color : Colors.grey.shade300,
                  borderRadius: BorderRadius.circular(2),
                ),
              ),
            );
          }),
        ),
        const SizedBox(height: 4),
        Text(
          'ความแข็งแกร่ง: $_label',
          style: TextStyle(fontSize: 12, color: _color),
        ),
      ],
    );
  }
}

// Demo Screen
class ValidatorsDemo extends StatefulWidget {
  const ValidatorsDemo({super.key});

  @override
  State<ValidatorsDemo> createState() => _ValidatorsDemoState();
}

class _ValidatorsDemoState extends State<ValidatorsDemo> {
  final _formKey = GlobalKey<FormState>();
  String _password = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Validators Demo')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          autovalidateMode: AutovalidateMode.onUserInteraction,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // Email
              TextFormField(
                decoration: const InputDecoration(
                  labelText: 'อีเมล',
                  border: OutlineInputBorder(),
                ),
                validator: Validators.email,
              ),

              const SizedBox(height: 16),

              // Thai Phone
              TextFormField(
                keyboardType: TextInputType.phone,
                decoration: const InputDecoration(
                  labelText: 'เบอร์โทร',
                  border: OutlineInputBorder(),
                  hintText: '0812345678',
                ),
                validator: Validators.thaiPhone,
              ),

              const SizedBox(height: 16),

              // National ID
              TextFormField(
                keyboardType: TextInputType.number,
                decoration: const InputDecoration(
                  labelText: 'เลขบัตรประชาชน',
                  border: OutlineInputBorder(),
                  hintText: 'XXXXXXXXXXXXX',
                ),
                validator: Validators.nationalId,
              ),

              const SizedBox(height: 16),

              // Password พร้อม strength indicator
              TextFormField(
                obscureText: true,
                decoration: const InputDecoration(
                  labelText: 'รหัสผ่าน',
                  border: OutlineInputBorder(),
                ),
                onChanged: (v) => setState(() => _password = v),
                validator: Validators.password,
              ),
              const SizedBox(height: 8),
              PasswordStrengthIndicator(password: _password),

              const SizedBox(height: 16),

              // Composed validators
              TextFormField(
                decoration: const InputDecoration(
                  labelText: 'ชื่อผู้ใช้ (4-20 ตัวอักษร)',
                  border: OutlineInputBorder(),
                ),
                validator: Validators.compose([
                  Validators.required,
                  Validators.minLength(4),
                  Validators.maxLength(20),
                ]),
              ),

              const SizedBox(height: 24),

              ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState?.validate() == true) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(content: Text('Form ถูกต้องทั้งหมด!')),
                    );
                  }
                },
                child: const Text('ตรวจสอบ'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 74: Checkbox, Radio, Switch

```dart
import 'package:flutter/material.dart';

class SelectionControlsDemo extends StatefulWidget {
  const SelectionControlsDemo({super.key});

  @override
  State<SelectionControlsDemo> createState() =>
      _SelectionControlsDemoState();
}

class _SelectionControlsDemoState extends State<SelectionControlsDemo> {
  // Checkbox states
  bool _check1 = false;
  bool _check2 = true;
  bool? _tristate = null; // null = indeterminate

  // Radio
  String _selectedGender = 'male';
  int _selectedPayment = 1;

  // Switch
  bool _notifications = true;
  bool _darkMode = false;
  bool _autoUpdate = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Selection Controls')),
      body: SingleChildScrollView(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // ========= CHECKBOX =========
            const _SectionHeader('Checkbox'),

            CheckboxListTile(
              title: const Text('รับข่าวสารผ่านอีเมล'),
              subtitle: const Text('เราจะส่งข่าวสารล่าสุดให้คุณ'),
              value: _check1,
              onChanged: (v) => setState(() => _check1 = v!),
              secondary: const Icon(Icons.email_outlined),
            ),

            CheckboxListTile(
              title: const Text('ยอมรับนโยบายความเป็นส่วนตัว'),
              value: _check2,
              onChanged: (v) => setState(() => _check2 = v!),
              activeColor: Colors.green,
              checkColor: Colors.white,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(4),
              ),
            ),

            // Tristate Checkbox
            ListTile(
              title: const Text('Tristate Checkbox'),
              subtitle: Text(
                  'ค่า: ${_tristate == null ? 'indeterminate' : _tristate}'),
              leading: Checkbox(
                tristate: true,
                value: _tristate,
                onChanged: (v) => setState(() => _tristate = v),
              ),
            ),

            // Custom Checkbox Row
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('เลือกความสนใจ:',
                      style: TextStyle(fontWeight: FontWeight.bold)),
                  const SizedBox(height: 8),
                  _CheckboxGroup(),
                ],
              ),
            ),

            const Divider(),

            // ========= RADIO =========
            const _SectionHeader('Radio'),

            // เพศ
            Padding(
              padding:
                  const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('เพศ:',
                      style: TextStyle(fontWeight: FontWeight.bold)),
                  Row(
                    children: [
                      Expanded(
                        child: RadioListTile<String>(
                          title: const Text('ชาย'),
                          value: 'male',
                          groupValue: _selectedGender,
                          onChanged: (v) =>
                              setState(() => _selectedGender = v!),
                        ),
                      ),
                      Expanded(
                        child: RadioListTile<String>(
                          title: const Text('หญิง'),
                          value: 'female',
                          groupValue: _selectedGender,
                          onChanged: (v) =>
                              setState(() => _selectedGender = v!),
                        ),
                      ),
                    ],
                  ),
                  RadioListTile<String>(
                    title: const Text('ไม่ระบุ'),
                    value: 'other',
                    groupValue: _selectedGender,
                    onChanged: (v) =>
                        setState(() => _selectedGender = v!),
                  ),
                ],
              ),
            ),

            // วิธีชำระเงิน
            Padding(
              padding:
                  const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('วิธีชำระเงิน:',
                      style: TextStyle(fontWeight: FontWeight.bold)),
                  const SizedBox(height: 8),
                  ...[
                    (1, 'บัตรเครดิต/เดบิต', Icons.credit_card),
                    (2, 'พร้อมเพย์', Icons.qr_code),
                    (3, 'โมบายแบงกิ้ง', Icons.phone_android),
                    (4, 'ปลายทาง', Icons.local_shipping),
                  ].map((option) => Card(
                    margin: const EdgeInsets.only(bottom: 8),
                    child: RadioListTile<int>(
                      title: Text(option.$2),
                      value: option.$1,
                      groupValue: _selectedPayment,
                      onChanged: (v) =>
                          setState(() => _selectedPayment = v!),
                      secondary: Icon(option.$3),
                      activeColor: Colors.blue,
                    ),
                  )),
                ],
              ),
            ),

            const Divider(),

            // ========= SWITCH =========
            const _SectionHeader('Switch'),

            SwitchListTile(
              title: const Text('การแจ้งเตือน'),
              subtitle:
                  Text(_notifications ? 'เปิดอยู่' : 'ปิดอยู่'),
              value: _notifications,
              onChanged: (v) => setState(() => _notifications = v),
              secondary: Icon(
                _notifications
                    ? Icons.notifications_active
                    : Icons.notifications_off,
                color: _notifications ? Colors.blue : Colors.grey,
              ),
            ),

            SwitchListTile(
              title: const Text('โหมดมืด'),
              value: _darkMode,
              onChanged: (v) => setState(() => _darkMode = v),
              secondary: Icon(
                _darkMode ? Icons.dark_mode : Icons.light_mode,
                color: _darkMode ? Colors.indigo : Colors.orange,
              ),
              activeColor: Colors.indigo,
            ),

            SwitchListTile.adaptive(  // adaptive: iOS ใช้ CupertinoSwitch
              title: const Text('อัปเดตอัตโนมัติ'),
              subtitle: const Text('(Adaptive Switch)'),
              value: _autoUpdate,
              onChanged: (v) => setState(() => _autoUpdate = v),
            ),

            const SizedBox(height: 16),
          ],
        ),
      ),
    );
  }
}

class _SectionHeader extends StatelessWidget {
  final String title;
  const _SectionHeader(this.title);

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.fromLTRB(16, 16, 16, 4),
      child: Text(
        title,
        style: const TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.bold,
          color: Colors.deepPurple,
        ),
      ),
    );
  }
}

class _CheckboxGroup extends StatefulWidget {
  @override
  State<_CheckboxGroup> createState() => _CheckboxGroupState();
}

class _CheckboxGroupState extends State<_CheckboxGroup> {
  final Map<String, bool> _interests = {
    'เทคโนโลยี': false,
    'กีฬา': true,
    'ดนตรี': false,
    'ท่องเที่ยว': true,
    'อาหาร': false,
    'ศิลปะ': true,
  };

  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 8,
      runSpacing: 4,
      children: _interests.entries.map((entry) {
        return FilterChip(
          label: Text(entry.key),
          selected: entry.value,
          onSelected: (selected) =>
              setState(() => _interests[entry.key] = selected),
          selectedColor: Colors.deepPurple.shade100,
          checkmarkColor: Colors.deepPurple,
        );
      }).toList(),
    );
  }
}
```

---

## ขั้นตอนที่ 75: Slider & RangeSlider

```dart
import 'package:flutter/material.dart';

class SliderDemo extends StatefulWidget {
  const SliderDemo({super.key});

  @override
  State<SliderDemo> createState() => _SliderDemoState();
}

class _SliderDemoState extends State<SliderDemo> {
  double _volume = 50;
  double _brightness = 0.7;
  double _rating = 3.5;
  RangeValues _priceRange = const RangeValues(500, 3000);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Slider & RangeSlider')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. Basic Slider
            const Text('ระดับเสียง:', style: TextStyle(fontWeight: FontWeight.bold)),
            Row(
              children: [
                Icon(
                  _volume == 0
                      ? Icons.volume_off
                      : _volume < 50
                          ? Icons.volume_down
                          : Icons.volume_up,
                  color: Colors.grey,
                ),
                Expanded(
                  child: Slider(
                    value: _volume,
                    min: 0,
                    max: 100,
                    divisions: 100,
                    label: '${_volume.toInt()}%',
                    onChanged: (v) => setState(() => _volume = v),
                  ),
                ),
                SizedBox(
                  width: 40,
                  child: Text('${_volume.toInt()}%'),
                ),
              ],
            ),

            const SizedBox(height: 16),

            // 2. Slider พร้อม step divisions
            const Text('ความสว่าง:', style: TextStyle(fontWeight: FontWeight.bold)),
            Row(
              children: [
                const Icon(Icons.brightness_low, color: Colors.grey),
                Expanded(
                  child: Slider(
                    value: _brightness,
                    min: 0.1,
                    max: 1.0,
                    divisions: 9,
                    label: '${(_brightness * 100).toInt()}%',
                    activeColor: Colors.amber,
                    inactiveColor: Colors.amber.shade100,
                    onChanged: (v) =>
                        setState(() => _brightness = v),
                  ),
                ),
                const Icon(Icons.brightness_high, color: Colors.amber),
              ],
            ),

            const SizedBox(height: 16),

            // 3. Rating Slider
            const Text('คะแนน:', style: TextStyle(fontWeight: FontWeight.bold)),
            Row(
              children: [
                Expanded(
                  child: Slider(
                    value: _rating,
                    min: 1,
                    max: 5,
                    divisions: 8,
                    label: _rating.toStringAsFixed(1),
                    activeColor: Colors.amber,
                    onChanged: (v) => setState(() => _rating = v),
                  ),
                ),
                Row(
                  children: [
                    const Icon(Icons.star, color: Colors.amber),
                    Text(_rating.toStringAsFixed(1),
                        style: const TextStyle(fontWeight: FontWeight.bold)),
                  ],
                ),
              ],
            ),

            const SizedBox(height: 24),

            // 4. RangeSlider
            const Text('ช่วงราคา:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text('฿${_priceRange.start.toInt()}'),
                Text('฿${_priceRange.end.toInt()}'),
              ],
            ),
            RangeSlider(
              values: _priceRange,
              min: 0,
              max: 10000,
              divisions: 100,
              labels: RangeLabels(
                '฿${_priceRange.start.toInt()}',
                '฿${_priceRange.end.toInt()}',
              ),
              onChanged: (values) =>
                  setState(() => _priceRange = values),
            ),
            Text(
              'ราคา ฿${_priceRange.start.toInt()} - ฿${_priceRange.end.toInt()}',
              style: const TextStyle(color: Colors.grey),
            ),

            const SizedBox(height: 24),

            // 5. Slider.adaptive
            const Text('Slider Adaptive:', style: TextStyle(fontWeight: FontWeight.bold)),
            Slider.adaptive(
              value: _volume,
              min: 0,
              max: 100,
              onChanged: (v) => setState(() => _volume = v),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 76: DropdownButton & DropdownMenu

```dart
import 'package:flutter/material.dart';

class DropdownDemo extends StatefulWidget {
  const DropdownDemo({super.key});

  @override
  State<DropdownDemo> createState() => _DropdownDemoState();
}

class _DropdownDemoState extends State<DropdownDemo> {
  String? _selectedProvince;
  String _selectedPayment = 'credit';
  String? _selectedCountry;
  String _dropdownMenuValue = '';

  final List<String> _provinces = [
    'กรุงเทพมหานคร', 'เชียงใหม่', 'ภูเก็ต', 'ขอนแก่น',
    'นครราชสีมา', 'อุดรธานี', 'สงขลา', 'ชลบุรี', 'สุราษฎร์ธานี',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dropdown')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. DropdownButton พื้นฐาน
            const Text('1. DropdownButton:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            DropdownButtonFormField<String>(
              value: _selectedProvince,
              hint: const Text('เลือกจังหวัด'),
              decoration: const InputDecoration(
                labelText: 'จังหวัด',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.location_on_outlined),
              ),
              items: _provinces.map((province) {
                return DropdownMenuItem(
                  value: province,
                  child: Text(province),
                );
              }).toList(),
              onChanged: (value) =>
                  setState(() => _selectedProvince = value),
              validator: (value) =>
                  value == null ? 'กรุณาเลือกจังหวัด' : null,
            ),

            if (_selectedProvince != null)
              Padding(
                padding: const EdgeInsets.only(top: 8),
                child: Text('เลือก: $_selectedProvince',
                    style: const TextStyle(color: Colors.grey)),
              ),

            const SizedBox(height: 24),

            // 2. Dropdown with icons
            const Text('2. Dropdown พร้อม Icon:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            DropdownButtonFormField<String>(
              value: _selectedPayment,
              decoration: const InputDecoration(
                labelText: 'วิธีชำระเงิน',
                border: OutlineInputBorder(),
              ),
              items: const [
                DropdownMenuItem(
                  value: 'credit',
                  child: Row(children: [
                    Icon(Icons.credit_card, size: 20),
                    SizedBox(width: 8),
                    Text('บัตรเครดิต'),
                  ]),
                ),
                DropdownMenuItem(
                  value: 'promptpay',
                  child: Row(children: [
                    Icon(Icons.qr_code, size: 20),
                    SizedBox(width: 8),
                    Text('พร้อมเพย์'),
                  ]),
                ),
                DropdownMenuItem(
                  value: 'bank',
                  child: Row(children: [
                    Icon(Icons.account_balance, size: 20),
                    SizedBox(width: 8),
                    Text('โอนเงิน'),
                  ]),
                ),
                DropdownMenuItem(
                  value: 'cod',
                  child: Row(children: [
                    Icon(Icons.local_shipping, size: 20),
                    SizedBox(width: 8),
                    Text('เก็บเงินปลายทาง'),
                  ]),
                ),
              ],
              onChanged: (v) => setState(() => _selectedPayment = v!),
            ),

            const SizedBox(height: 24),

            // 3. DropdownMenu (Material 3)
            const Text('3. DropdownMenu (M3):',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            DropdownMenu<String>(
              label: const Text('ประเทศ'),
              width: double.infinity,
              hintText: 'เลือกประเทศ',
              enableFilter: true,  // กรองได้
              enableSearch: true,
              leadingIcon: const Icon(Icons.flag_outlined),
              onSelected: (value) =>
                  setState(() => _selectedCountry = value),
              dropdownMenuEntries: [
                'ไทย', 'ญี่ปุ่น', 'เกาหลีใต้', 'สหรัฐอเมริกา',
                'สหราชอาณาจักร', 'ออสเตรเลีย', 'สิงคโปร์', 'จีน',
              ].map((country) {
                return DropdownMenuEntry(
                  value: country,
                  label: country,
                );
              }).toList(),
            ),

            if (_selectedCountry != null)
              Padding(
                padding: const EdgeInsets.only(top: 8),
                child: Text('เลือก: $_selectedCountry'),
              ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 77: DatePicker & TimePicker

```dart
import 'package:flutter/material.dart';

class DateTimePickerDemo extends StatefulWidget {
  const DateTimePickerDemo({super.key});

  @override
  State<DateTimePickerDemo> createState() => _DateTimePickerDemoState();
}

class _DateTimePickerDemoState extends State<DateTimePickerDemo> {
  DateTime? _selectedDate;
  TimeOfDay? _selectedTime;
  DateTimeRange? _dateRange;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('DatePicker & TimePicker')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            // 1. DatePicker
            Card(
              child: ListTile(
                leading: const Icon(Icons.calendar_today, color: Colors.blue),
                title: const Text('เลือกวันที่'),
                subtitle: Text(
                  _selectedDate != null
                      ? _formatDate(_selectedDate!)
                      : 'ยังไม่ได้เลือก',
                  style: TextStyle(
                    color: _selectedDate != null
                        ? Colors.black
                        : Colors.grey,
                  ),
                ),
                trailing: const Icon(Icons.arrow_forward_ios, size: 14),
                onTap: () => _pickDate(context),
              ),
            ),

            const SizedBox(height: 12),

            // 2. TimePicker
            Card(
              child: ListTile(
                leading: const Icon(Icons.access_time, color: Colors.green),
                title: const Text('เลือกเวลา'),
                subtitle: Text(
                  _selectedTime != null
                      ? _selectedTime!.format(context)
                      : 'ยังไม่ได้เลือก',
                  style: TextStyle(
                    color: _selectedTime != null
                        ? Colors.black
                        : Colors.grey,
                  ),
                ),
                trailing: const Icon(Icons.arrow_forward_ios, size: 14),
                onTap: () => _pickTime(context),
              ),
            ),

            const SizedBox(height: 12),

            // 3. DateRange Picker
            Card(
              child: ListTile(
                leading:
                    const Icon(Icons.date_range, color: Colors.orange),
                title: const Text('เลือกช่วงวันที่'),
                subtitle: Text(
                  _dateRange != null
                      ? '${_formatDate(_dateRange!.start)} - ${_formatDate(_dateRange!.end)}'
                      : 'ยังไม่ได้เลือก',
                  style: TextStyle(
                    color: _dateRange != null ? Colors.black : Colors.grey,
                  ),
                ),
                trailing: const Icon(Icons.arrow_forward_ios, size: 14),
                onTap: () => _pickDateRange(context),
              ),
            ),

            if (_selectedDate != null && _selectedTime != null) ...[
              const SizedBox(height: 24),
              Card(
                color: Colors.green.shade50,
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    children: [
                      const Icon(Icons.check_circle, color: Colors.green, size: 32),
                      const SizedBox(height: 8),
                      const Text('นัดหมาย:',
                          style: TextStyle(fontWeight: FontWeight.bold)),
                      Text(
                        '${_formatDate(_selectedDate!)} เวลา ${_selectedTime!.format(context)}',
                        style: const TextStyle(fontSize: 16),
                      ),
                    ],
                  ),
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }

  Future<void> _pickDate(BuildContext context) async {
    final now = DateTime.now();
    final picked = await showDatePicker(
      context: context,
      initialDate: _selectedDate ?? now,
      firstDate: DateTime(2020),
      lastDate: DateTime(2030),
      helpText: 'เลือกวันที่',
      cancelText: 'ยกเลิก',
      confirmText: 'ตกลง',
      locale: const Locale('th'),
      builder: (context, child) {
        return Theme(
          data: Theme.of(context).copyWith(
            colorScheme: Theme.of(context).colorScheme.copyWith(
              primary: Colors.blue,
            ),
          ),
          child: child!,
        );
      },
    );
    if (picked != null) setState(() => _selectedDate = picked);
  }

  Future<void> _pickTime(BuildContext context) async {
    final picked = await showTimePicker(
      context: context,
      initialTime: _selectedTime ?? TimeOfDay.now(),
      helpText: 'เลือกเวลา',
      cancelText: 'ยกเลิก',
      confirmText: 'ตกลง',
    );
    if (picked != null) setState(() => _selectedTime = picked);
  }

  Future<void> _pickDateRange(BuildContext context) async {
    final now = DateTime.now();
    final picked = await showDateRangePicker(
      context: context,
      firstDate: DateTime(2020),
      lastDate: DateTime(2030),
      initialDateRange: _dateRange,
      helpText: 'เลือกช่วงวันที่',
      cancelText: 'ยกเลิก',
      confirmText: 'ตกลง',
      saveText: 'บันทึก',
    );
    if (picked != null) setState(() => _dateRange = picked);
  }

  String _formatDate(DateTime date) {
    const months = [
      '', 'ม.ค.', 'ก.พ.', 'มี.ค.', 'เม.ย.',
      'พ.ค.', 'มิ.ย.', 'ก.ค.', 'ส.ค.',
      'ก.ย.', 'ต.ค.', 'พ.ย.', 'ธ.ค.',
    ];
    final buddhistYear = date.year + 543;
    return '${date.day} ${months[date.month]} $buddhistYear';
  }
}
```

---

## ขั้นตอนที่ 78: Autocomplete & SearchAnchor

```dart
import 'package:flutter/material.dart';

class AutocompleteDemo extends StatefulWidget {
  const AutocompleteDemo({super.key});

  @override
  State<AutocompleteDemo> createState() => _AutocompleteDemoState();
}

class _AutocompleteDemoState extends State<AutocompleteDemo> {
  final List<String> _allOptions = [
    'Flutter', 'Dart', 'React', 'Vue.js', 'Angular', 'SwiftUI',
    'Kotlin', 'Python', 'JavaScript', 'TypeScript', 'Rust', 'Go',
    'Firebase', 'Supabase', 'MongoDB', 'PostgreSQL', 'MySQL',
  ];

  String _selectedValue = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Autocomplete')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. Autocomplete พื้นฐาน
            const Text('1. Autocomplete:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Autocomplete<String>(
              optionsBuilder: (textEditingValue) {
                if (textEditingValue.text.isEmpty) {
                  return const [];
                }
                return _allOptions.where((option) =>
                    option.toLowerCase().contains(
                        textEditingValue.text.toLowerCase()));
              },
              onSelected: (value) =>
                  setState(() => _selectedValue = value),
              fieldViewBuilder: (context, controller, focusNode, onSubmit) {
                return TextField(
                  controller: controller,
                  focusNode: focusNode,
                  onSubmitted: (_) => onSubmit(),
                  decoration: const InputDecoration(
                    labelText: 'ค้นหาเทคโนโลยี',
                    prefixIcon: Icon(Icons.search),
                    border: OutlineInputBorder(),
                  ),
                );
              },
              optionsViewBuilder: (context, onSelected, options) {
                return Align(
                  alignment: Alignment.topLeft,
                  child: Material(
                    elevation: 4,
                    borderRadius: BorderRadius.circular(8),
                    child: ConstrainedBox(
                      constraints: const BoxConstraints(maxHeight: 200),
                      child: ListView.builder(
                        shrinkWrap: true,
                        itemCount: options.length,
                        itemBuilder: (context, index) {
                          final option = options.elementAt(index);
                          return ListTile(
                            leading: const Icon(Icons.code, size: 20),
                            title: Text(option),
                            onTap: () => onSelected(option),
                          );
                        },
                      ),
                    ),
                  ),
                );
              },
            ),

            if (_selectedValue.isNotEmpty)
              Padding(
                padding: const EdgeInsets.only(top: 8),
                child: Text('เลือก: $_selectedValue',
                    style: const TextStyle(color: Colors.grey)),
              ),

            const SizedBox(height: 24),

            // 2. SearchAnchor (Material 3)
            const Text('2. SearchAnchor (M3):', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            SearchAnchor(
              builder: (context, controller) {
                return SearchBar(
                  controller: controller,
                  hintText: 'ค้นหา...',
                  onTap: () => controller.openView(),
                  onChanged: (_) => controller.openView(),
                  leading: const Icon(Icons.search),
                  trailing: [
                    IconButton(
                      icon: const Icon(Icons.filter_list),
                      onPressed: () {},
                    ),
                  ],
                );
              },
              suggestionsBuilder: (context, controller) {
                final query = controller.text.toLowerCase();
                final filtered = query.isEmpty
                    ? _allOptions
                    : _allOptions
                        .where((o) => o.toLowerCase().contains(query))
                        .toList();

                return filtered.map((option) {
                  return ListTile(
                    leading: const Icon(Icons.code),
                    title: Text(option),
                    onTap: () {
                      controller.closeView(option);
                      setState(() => _selectedValue = option);
                    },
                  );
                }).toList();
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 79: FocusNode & TextEditingController

```dart
import 'package:flutter/material.dart';

class FocusControllerDemo extends StatefulWidget {
  const FocusControllerDemo({super.key});

  @override
  State<FocusControllerDemo> createState() => _FocusControllerDemoState();
}

class _FocusControllerDemoState extends State<FocusControllerDemo> {
  // TextEditingController สำหรับอ่าน/เขียนค่า
  final _nameController = TextEditingController(text: '');
  final _emailController = TextEditingController();
  final _phoneController = TextEditingController();
  final _messageController = TextEditingController();

  // FocusNode สำหรับควบคุม focus
  final _nameFocus = FocusNode();
  final _emailFocus = FocusNode();
  final _phoneFocus = FocusNode();
  final _messageFocus = FocusNode();

  // Track focused field
  String _focusedField = 'none';

  @override
  void initState() {
    super.initState();

    // Listen to controller changes
    _nameController.addListener(() {
      // ทำอะไรก็ได้เมื่อค่าเปลี่ยน
      setState(() {});
    });

    // Listen to focus changes
    _nameFocus.addListener(() {
      if (_nameFocus.hasFocus) {
        setState(() => _focusedField = 'name');
      }
    });
    _emailFocus.addListener(() {
      if (_emailFocus.hasFocus) {
        setState(() => _focusedField = 'email');
      }
    });
    _phoneFocus.addListener(() {
      if (_phoneFocus.hasFocus) {
        setState(() => _focusedField = 'phone');
      }
    });
    _messageFocus.addListener(() {
      if (_messageFocus.hasFocus) {
        setState(() => _focusedField = 'message');
      }
    });
  }

  @override
  void dispose() {
    // Dispose ทั้ง controllers และ focus nodes
    _nameController.dispose();
    _emailController.dispose();
    _phoneController.dispose();
    _messageController.dispose();
    _nameFocus.dispose();
    _emailFocus.dispose();
    _phoneFocus.dispose();
    _messageFocus.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('FocusNode & Controller')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Focus indicator
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: Colors.blue.shade50,
                borderRadius: BorderRadius.circular(8),
              ),
              child: Text('กำลัง focus: $_focusedField'),
            ),

            const SizedBox(height: 16),

            // ชื่อ - auto focus
            TextField(
              controller: _nameController,
              focusNode: _nameFocus,
              autofocus: true,  // auto focus เมื่อเปิดหน้า
              textInputAction: TextInputAction.next,
              decoration: InputDecoration(
                labelText: 'ชื่อ',
                border: const OutlineInputBorder(),
                suffixText: '${_nameController.text.length}/50',
              ),
              maxLength: 50,
              onSubmitted: (_) =>
                  FocusScope.of(context).requestFocus(_emailFocus),
            ),

            const SizedBox(height: 16),

            // อีเมล
            TextField(
              controller: _emailController,
              focusNode: _emailFocus,
              keyboardType: TextInputType.emailAddress,
              textInputAction: TextInputAction.next,
              decoration: const InputDecoration(
                labelText: 'อีเมล',
                border: OutlineInputBorder(),
              ),
              onSubmitted: (_) =>
                  FocusScope.of(context).requestFocus(_phoneFocus),
            ),

            const SizedBox(height: 16),

            // เบอร์โทร
            TextField(
              controller: _phoneController,
              focusNode: _phoneFocus,
              keyboardType: TextInputType.phone,
              textInputAction: TextInputAction.next,
              decoration: const InputDecoration(
                labelText: 'เบอร์โทร',
                border: OutlineInputBorder(),
              ),
              onSubmitted: (_) =>
                  FocusScope.of(context).requestFocus(_messageFocus),
            ),

            const SizedBox(height: 16),

            // ข้อความ
            TextField(
              controller: _messageController,
              focusNode: _messageFocus,
              maxLines: 3,
              textInputAction: TextInputAction.done,
              decoration: const InputDecoration(
                labelText: 'ข้อความ',
                border: OutlineInputBorder(),
                alignLabelWithHint: true,
              ),
              onSubmitted: (_) =>
                  FocusScope.of(context).unfocus(),
            ),

            const SizedBox(height: 16),

            // Controller operations
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      // อ่านค่า
                      final name = _nameController.text;
                      final email = _emailController.text;
                      showDialog(
                        context: context,
                        builder: (_) => AlertDialog(
                          title: const Text('ข้อมูล'),
                          content: Text('ชื่อ: $name\nอีเมล: $email'),
                          actions: [
                            TextButton(
                              onPressed: () => Navigator.pop(context),
                              child: const Text('ตกลง'),
                            ),
                          ],
                        ),
                      );
                    },
                    child: const Text('อ่านค่า'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: OutlinedButton(
                    onPressed: () {
                      // ล้างค่าทั้งหมด
                      _nameController.clear();
                      _emailController.clear();
                      _phoneController.clear();
                      _messageController.clear();
                      // set focus กลับไปที่แรก
                      _nameFocus.requestFocus();
                    },
                    child: const Text('ล้าง'),
                  ),
                ),
              ],
            ),

            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () {
                // Set value programmatically
                _nameController.value = TextEditingValue(
                  text: 'สมชาย ใจดี',
                  selection: TextSelection.fromPosition(
                    TextPosition(offset: 'สมชาย ใจดี'.length),
                  ),
                );
                _emailController.text = 'somchai@example.com';
              },
              child: const Text('กรอกข้อมูลตัวอย่าง'),
            ),

            const SizedBox(height: 16),

            // แสดงค่าปัจจุบัน
            Card(
              child: Padding(
                padding: const EdgeInsets.all(12),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text('ค่าปัจจุบัน:',
                        style: TextStyle(fontWeight: FontWeight.bold)),
                    Text('ชื่อ: ${_nameController.text}'),
                    Text('อีเมล: ${_emailController.text}'),
                    Text('เบอร์: ${_phoneController.text}'),
                    Text('ข้อความ: ${_messageController.text}'),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 80: Workshop - Registration Form

สร้าง Registration Form ที่สมบูรณ์

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const RegistrationApp());
}

class RegistrationApp extends StatelessWidget {
  const RegistrationApp({super.key});
  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Registration',
        theme: ThemeData(
          useMaterial3: true,
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        ),
        home: const RegistrationScreen(),
      );
}

class RegistrationScreen extends StatefulWidget {
  const RegistrationScreen({super.key});

  @override
  State<RegistrationScreen> createState() => _RegistrationScreenState();
}

class _RegistrationScreenState extends State<RegistrationScreen> {
  final _formKey = GlobalKey<FormState>();
  int _currentStep = 0;

  // Controllers
  final _firstNameCtrl = TextEditingController();
  final _lastNameCtrl = TextEditingController();
  final _emailCtrl = TextEditingController();
  final _phoneCtrl = TextEditingController();
  final _passwordCtrl = TextEditingController();
  final _confirmPasswordCtrl = TextEditingController();
  final _bioCtrl = TextEditingController();

  // State values
  bool _showPassword = false;
  bool _showConfirmPassword = false;
  String _gender = 'male';
  DateTime? _birthDate;
  String? _province;
  String? _occupation;
  bool _agreeTerms = false;
  bool _receiveNews = false;
  double _experience = 1.0;
  List<String> _skills = [];

  final List<String> _provinces = [
    'กรุงเทพมหานคร', 'เชียงใหม่', 'ภูเก็ต', 'ขอนแก่น',
    'นครราชสีมา', 'อุดรธานี', 'สงขลา', 'ชลบุรี',
  ];

  final List<String> _occupations = [
    'นักพัฒนาซอฟต์แวร์', 'นักออกแบบ UX/UI', 'นักการตลาด',
    'นักบริหาร', 'นักเรียน/นักศึกษา', 'อื่นๆ',
  ];

  final List<String> _allSkills = [
    'Flutter', 'Dart', 'React', 'Vue.js', 'Node.js',
    'Python', 'Firebase', 'AWS', 'Git', 'Figma',
  ];

  @override
  void dispose() {
    _firstNameCtrl.dispose();
    _lastNameCtrl.dispose();
    _emailCtrl.dispose();
    _phoneCtrl.dispose();
    _passwordCtrl.dispose();
    _confirmPasswordCtrl.dispose();
    _bioCtrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('สมัครสมาชิก'),
        leading: _currentStep > 0
            ? IconButton(
                icon: const Icon(Icons.arrow_back),
                onPressed: () =>
                    setState(() => _currentStep--),
              )
            : null,
      ),
      body: Column(
        children: [
          // Step Indicator
          _buildStepIndicator(),

          // Form Content
          Expanded(
            child: SingleChildScrollView(
              padding: const EdgeInsets.all(20),
              child: Form(
                key: _formKey,
                autovalidateMode: AutovalidateMode.onUserInteraction,
                child: _buildCurrentStep(),
              ),
            ),
          ),

          // Bottom Buttons
          _buildBottomButtons(),
        ],
      ),
    );
  }

  Widget _buildStepIndicator() {
    final steps = ['ข้อมูลส่วนตัว', 'ข้อมูลบัญชี', 'ความสนใจ'];
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white,
        boxShadow: [
          BoxShadow(
            color: Colors.grey.withOpacity(0.1),
            blurRadius: 4,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Row(
        children: steps.asMap().entries.map((entry) {
          final index = entry.key;
          final label = entry.value;
          final isActive = index == _currentStep;
          final isCompleted = index < _currentStep;

          return Expanded(
            child: Row(
              children: [
                Column(
                  children: [
                    Container(
                      width: 32,
                      height: 32,
                      decoration: BoxDecoration(
                        color: isCompleted
                            ? Colors.green
                            : isActive
                                ? Theme.of(context).colorScheme.primary
                                : Colors.grey.shade300,
                        shape: BoxShape.circle,
                      ),
                      child: Icon(
                        isCompleted ? Icons.check : Icons.circle,
                        color: Colors.white,
                        size: 16,
                      ),
                    ),
                    const SizedBox(height: 4),
                    Text(
                      label,
                      style: TextStyle(
                        fontSize: 10,
                        color: isActive
                            ? Theme.of(context).colorScheme.primary
                            : Colors.grey,
                        fontWeight: isActive
                            ? FontWeight.bold
                            : FontWeight.normal,
                      ),
                    ),
                  ],
                ),
                if (index < steps.length - 1)
                  Expanded(
                    child: Container(
                      height: 1,
                      color: index < _currentStep
                          ? Colors.green
                          : Colors.grey.shade300,
                      margin: const EdgeInsets.only(bottom: 20),
                    ),
                  ),
              ],
            ),
          );
        }).toList(),
      ),
    );
  }

  Widget _buildCurrentStep() {
    switch (_currentStep) {
      case 0:
        return _buildPersonalInfoStep();
      case 1:
        return _buildAccountStep();
      case 2:
        return _buildInterestsStep();
      default:
        return const SizedBox.shrink();
    }
  }

  Widget _buildPersonalInfoStep() {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Text('ข้อมูลส่วนตัว',
            style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
        const SizedBox(height: 4),
        const Text('กรุณากรอกข้อมูลส่วนตัวของคุณ',
            style: TextStyle(color: Colors.grey)),
        const SizedBox(height: 24),

        // ชื่อ - นามสกุล
        Row(
          children: [
            Expanded(
              child: TextFormField(
                controller: _firstNameCtrl,
                textInputAction: TextInputAction.next,
                decoration: const InputDecoration(
                  labelText: 'ชื่อ *',
                  border: OutlineInputBorder(),
                ),
                validator: (v) =>
                    v?.isEmpty == true ? 'กรุณากรอกชื่อ' : null,
              ),
            ),
            const SizedBox(width: 12),
            Expanded(
              child: TextFormField(
                controller: _lastNameCtrl,
                textInputAction: TextInputAction.next,
                decoration: const InputDecoration(
                  labelText: 'นามสกุล *',
                  border: OutlineInputBorder(),
                ),
                validator: (v) =>
                    v?.isEmpty == true ? 'กรุณากรอกนามสกุล' : null,
              ),
            ),
          ],
        ),

        const SizedBox(height: 16),

        // เพศ
        const Text('เพศ:', style: TextStyle(fontWeight: FontWeight.bold)),
        const SizedBox(height: 8),
        Row(
          children: [
            Expanded(
              child: RadioListTile<String>(
                title: const Text('ชาย'),
                value: 'male',
                groupValue: _gender,
                onChanged: (v) => setState(() => _gender = v!),
                contentPadding: EdgeInsets.zero,
              ),
            ),
            Expanded(
              child: RadioListTile<String>(
                title: const Text('หญิง'),
                value: 'female',
                groupValue: _gender,
                onChanged: (v) => setState(() => _gender = v!),
                contentPadding: EdgeInsets.zero,
              ),
            ),
          ],
        ),

        const SizedBox(height: 16),

        // วันเกิด
        InkWell(
          onTap: () => _pickBirthDate(),
          child: InputDecorator(
            decoration: const InputDecoration(
              labelText: 'วันเกิด',
              border: OutlineInputBorder(),
              suffixIcon: Icon(Icons.calendar_today),
            ),
            child: Text(
              _birthDate != null
                  ? '${_birthDate!.day}/${_birthDate!.month}/${_birthDate!.year + 543}'
                  : 'เลือกวันเกิด',
              style: TextStyle(
                color: _birthDate != null
                    ? Colors.black
                    : Colors.grey.shade600,
              ),
            ),
          ),
        ),

        const SizedBox(height: 16),

        // จังหวัด
        DropdownButtonFormField<String>(
          value: _province,
          hint: const Text('เลือกจังหวัด'),
          decoration: const InputDecoration(
            labelText: 'จังหวัด *',
            border: OutlineInputBorder(),
          ),
          items: _provinces
              .map((p) => DropdownMenuItem(value: p, child: Text(p)))
              .toList(),
          onChanged: (v) => setState(() => _province = v),
          validator: (v) =>
              v == null ? 'กรุณาเลือกจังหวัด' : null,
        ),

        const SizedBox(height: 16),

        // อาชีพ
        DropdownButtonFormField<String>(
          value: _occupation,
          hint: const Text('เลือกอาชีพ'),
          decoration: const InputDecoration(
            labelText: 'อาชีพ',
            border: OutlineInputBorder(),
          ),
          items: _occupations
              .map((o) => DropdownMenuItem(value: o, child: Text(o)))
              .toList(),
          onChanged: (v) => setState(() => _occupation = v),
        ),

        const SizedBox(height: 16),

        // ประวัติย่อ
        TextFormField(
          controller: _bioCtrl,
          maxLines: 3,
          maxLength: 200,
          decoration: const InputDecoration(
            labelText: 'แนะนำตัวสั้นๆ',
            border: OutlineInputBorder(),
            alignLabelWithHint: true,
          ),
        ),
      ],
    );
  }

  Widget _buildAccountStep() {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Text('ข้อมูลบัญชี',
            style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
        const SizedBox(height: 4),
        const Text('สร้างบัญชีผู้ใช้ของคุณ',
            style: TextStyle(color: Colors.grey)),
        const SizedBox(height: 24),

        // อีเมล
        TextFormField(
          controller: _emailCtrl,
          keyboardType: TextInputType.emailAddress,
          textInputAction: TextInputAction.next,
          decoration: const InputDecoration(
            labelText: 'อีเมล *',
            prefixIcon: Icon(Icons.email_outlined),
            border: OutlineInputBorder(),
          ),
          validator: (v) {
            if (v?.isEmpty == true) return 'กรุณากรอกอีเมล';
            if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$')
                .hasMatch(v!)) {
              return 'รูปแบบอีเมลไม่ถูกต้อง';
            }
            return null;
          },
        ),

        const SizedBox(height: 16),

        // เบอร์โทร
        TextFormField(
          controller: _phoneCtrl,
          keyboardType: TextInputType.phone,
          textInputAction: TextInputAction.next,
          inputFormatters: [
            FilteringTextInputFormatter.digitsOnly,
            LengthLimitingTextInputFormatter(10),
          ],
          decoration: const InputDecoration(
            labelText: 'เบอร์โทร *',
            prefixIcon: Icon(Icons.phone_outlined),
            border: OutlineInputBorder(),
            hintText: '0812345678',
          ),
          validator: (v) {
            if (v?.isEmpty == true) return 'กรุณากรอกเบอร์โทร';
            if (v!.length < 9) return 'เบอร์โทรไม่ถูกต้อง';
            return null;
          },
        ),

        const SizedBox(height: 16),

        // รหัสผ่าน
        TextFormField(
          controller: _passwordCtrl,
          obscureText: !_showPassword,
          textInputAction: TextInputAction.next,
          decoration: InputDecoration(
            labelText: 'รหัสผ่าน *',
            prefixIcon: const Icon(Icons.lock_outline),
            suffixIcon: IconButton(
              icon: Icon(
                  _showPassword ? Icons.visibility : Icons.visibility_off),
              onPressed: () =>
                  setState(() => _showPassword = !_showPassword),
            ),
            border: const OutlineInputBorder(),
            helperText: 'อย่างน้อย 8 ตัว มีตัวใหญ่ ตัวเล็ก และตัวเลข',
          ),
          validator: (v) {
            if (v?.isEmpty == true) return 'กรุณากรอกรหัสผ่าน';
            if (v!.length < 8) return 'รหัสผ่านต้องมีอย่างน้อย 8 ตัว';
            if (!v.contains(RegExp(r'[A-Z]'))) {
              return 'ต้องมีตัวพิมพ์ใหญ่';
            }
            if (!v.contains(RegExp(r'[0-9]'))) {
              return 'ต้องมีตัวเลข';
            }
            return null;
          },
        ),

        const SizedBox(height: 8),

        // Password strength
        ValueListenableBuilder(
          valueListenable: _passwordCtrl,
          builder: (context, value, _) {
            final p = value.text;
            int strength = 0;
            if (p.length >= 8) strength++;
            if (p.contains(RegExp(r'[A-Z]'))) strength++;
            if (p.contains(RegExp(r'[a-z]'))) strength++;
            if (p.contains(RegExp(r'[0-9]'))) strength++;
            if (p.contains(RegExp(r'[!@#\$]'))) strength++;

            if (p.isEmpty) return const SizedBox.shrink();

            final colors = [Colors.red, Colors.red, Colors.orange,
                Colors.yellow.shade700, Colors.lightGreen, Colors.green];
            final labels = ['', 'อ่อนมาก', 'อ่อน', 'ปานกลาง',
                'แข็งแกร่ง', 'แข็งแกร่งมาก'];

            return Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Row(
                  children: List.generate(5, (i) => Expanded(
                    child: Container(
                      margin: const EdgeInsets.only(right: 4),
                      height: 4,
                      decoration: BoxDecoration(
                        color: i < strength
                            ? colors[strength]
                            : Colors.grey.shade200,
                        borderRadius: BorderRadius.circular(2),
                      ),
                    ),
                  )),
                ),
                const SizedBox(height: 4),
                Text(
                  labels[strength],
                  style: TextStyle(
                      fontSize: 11, color: colors[strength]),
                ),
              ],
            );
          },
        ),

        const SizedBox(height: 16),

        // ยืนยันรหัสผ่าน
        TextFormField(
          controller: _confirmPasswordCtrl,
          obscureText: !_showConfirmPassword,
          decoration: InputDecoration(
            labelText: 'ยืนยันรหัสผ่าน *',
            prefixIcon: const Icon(Icons.lock_outline),
            suffixIcon: IconButton(
              icon: Icon(_showConfirmPassword
                  ? Icons.visibility
                  : Icons.visibility_off),
              onPressed: () => setState(
                  () => _showConfirmPassword = !_showConfirmPassword),
            ),
            border: const OutlineInputBorder(),
          ),
          validator: (v) {
            if (v != _passwordCtrl.text) {
              return 'รหัสผ่านไม่ตรงกัน';
            }
            return null;
          },
        ),
      ],
    );
  }

  Widget _buildInterestsStep() {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Text('ความสนใจ',
            style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
        const SizedBox(height: 4),
        const Text('บอกเราเพิ่มเติมเกี่ยวกับความสนใจของคุณ',
            style: TextStyle(color: Colors.grey)),
        const SizedBox(height: 24),

        // ทักษะ
        const Text('ทักษะที่มี:',
            style: TextStyle(fontWeight: FontWeight.bold)),
        const SizedBox(height: 8),
        Wrap(
          spacing: 8,
          runSpacing: 8,
          children: _allSkills.map((skill) {
            final isSelected = _skills.contains(skill);
            return FilterChip(
              label: Text(skill),
              selected: isSelected,
              onSelected: (selected) {
                setState(() {
                  if (selected) {
                    _skills.add(skill);
                  } else {
                    _skills.remove(skill);
                  }
                });
              },
            );
          }).toList(),
        ),

        const SizedBox(height: 24),

        // ประสบการณ์
        const Text('ประสบการณ์:',
            style: TextStyle(fontWeight: FontWeight.bold)),
        Slider(
          value: _experience,
          min: 0,
          max: 10,
          divisions: 10,
          label: _experience == 0
              ? 'ไม่มี'
              : '${_experience.toInt()} ปี',
          onChanged: (v) => setState(() => _experience = v),
        ),
        Center(
          child: Text(
            _experience == 0
                ? 'ไม่มีประสบการณ์'
                : 'ประสบการณ์ ${_experience.toInt()} ปี',
            style: const TextStyle(fontWeight: FontWeight.w500),
          ),
        ),

        const SizedBox(height: 24),

        // เงื่อนไข
        CheckboxListTile(
          title: const Text.rich(
            TextSpan(children: [
              TextSpan(text: 'ยอมรับ '),
              TextSpan(
                text: 'เงื่อนไขการใช้บริการ',
                style: TextStyle(
                  color: Colors.blue,
                  decoration: TextDecoration.underline,
                ),
              ),
              TextSpan(text: ' และ '),
              TextSpan(
                text: 'นโยบายความเป็นส่วนตัว',
                style: TextStyle(
                  color: Colors.blue,
                  decoration: TextDecoration.underline,
                ),
              ),
              TextSpan(text: ' *'),
            ]),
          ),
          value: _agreeTerms,
          onChanged: (v) => setState(() => _agreeTerms = v!),
          controlAffinity: ListTileControlAffinity.leading,
          contentPadding: EdgeInsets.zero,
        ),

        CheckboxListTile(
          title: const Text('รับข่าวสารและโปรโมชั่น'),
          value: _receiveNews,
          onChanged: (v) => setState(() => _receiveNews = v!),
          controlAffinity: ListTileControlAffinity.leading,
          contentPadding: EdgeInsets.zero,
        ),
      ],
    );
  }

  Widget _buildBottomButtons() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white,
        boxShadow: [
          BoxShadow(
            color: Colors.grey.withOpacity(0.1),
            blurRadius: 4,
            offset: const Offset(0, -2),
          ),
        ],
      ),
      child: Row(
        children: [
          if (_currentStep > 0)
            Expanded(
              child: OutlinedButton(
                onPressed: () =>
                    setState(() => _currentStep--),
                child: const Text('ย้อนกลับ'),
              ),
            ),
          if (_currentStep > 0) const SizedBox(width: 12),
          Expanded(
            flex: 2,
            child: ElevatedButton(
              onPressed: _handleNext,
              child: Text(_currentStep < 2 ? 'ถัดไป' : 'สมัครสมาชิก'),
            ),
          ),
        ],
      ),
    );
  }

  void _handleNext() {
    if (_currentStep < 2) {
      if (_formKey.currentState?.validate() == true) {
        setState(() => _currentStep++);
      }
    } else {
      // Step สุดท้าย
      if (!_agreeTerms) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('กรุณายอมรับเงื่อนไขการใช้บริการ'),
            backgroundColor: Colors.red,
          ),
        );
        return;
      }

      if (_formKey.currentState?.validate() == true) {
        _showSuccessDialog();
      }
    }
  }

  void _showSuccessDialog() {
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (context) => AlertDialog(
        icon: const Icon(Icons.check_circle, color: Colors.green, size: 64),
        title: const Text('สมัครสมาชิกสำเร็จ!'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('ยินดีต้อนรับ ${_firstNameCtrl.text} ${_lastNameCtrl.text}'),
            const SizedBox(height: 8),
            Text('อีเมล: ${_emailCtrl.text}'),
            if (_skills.isNotEmpty)
              Text('ทักษะ: ${_skills.join(", ")}'),
          ],
        ),
        actions: [
          ElevatedButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('เริ่มใช้งาน'),
          ),
        ],
      ),
    );
  }

  Future<void> _pickBirthDate() async {
    final now = DateTime.now();
    final picked = await showDatePicker(
      context: context,
      initialDate: _birthDate ?? DateTime(now.year - 20),
      firstDate: DateTime(1950),
      lastDate: DateTime(now.year - 13),
      helpText: 'เลือกวันเกิด',
    );
    if (picked != null) setState(() => _birthDate = picked);
  }
}
```

---

## สรุป (Summary)

ใน Part 08 เราเรียนเรื่อง Forms และ Input:

| Widget/Concept | การใช้งาน |
|---------------|-----------|
| `TextField` | รับข้อความ พร้อม inputFormatters |
| `TextFormField` | TextField + validation ใน Form |
| `Form` + `GlobalKey<FormState>` | รวม field หลายตัว validate พร้อมกัน |
| `Validators` | ตรวจสอบ email, phone, nationalId, password |
| `Checkbox/Radio/Switch` | เลือก boolean, เลือกตัวเดียว, toggle |
| `Slider/RangeSlider` | เลือกค่าจาก range |
| `DropdownButton/DropdownMenu` | เลือกจากรายการ |
| `DatePicker/TimePicker` | เลือกวันและเวลา |
| `Autocomplete/SearchAnchor` | ค้นหาพร้อมแนะนำ |
| `FocusNode` | ควบคุม focus ระหว่าง fields |
| `TextEditingController` | อ่าน/เขียนค่า TextField |

---

## แบบฝึกหัด (Exercises)

1. **Login Form**: สร้าง login form พร้อม email/password validation, remember me checkbox, forgot password link

2. **Payment Form**: สร้าง form รับข้อมูลบัตรเครดิต พร้อม number formatting (XXXX-XXXX-XXXX-XXXX), expiry date, CVV

3. **Survey Form**: สร้างแบบสอบถาม 5 ข้อ ใช้ radio, checkbox, slider, dropdown และ text field

4. **Profile Editor**: สร้าง profile edit screen พร้อม avatar picker, name, bio, birthday, skills

5. **Multi-step Checkout**: สร้าง checkout process 3 ขั้นตอน: ที่อยู่, การชำระเงิน, ยืนยัน

---

[← Part 07: Material Design & Theming](part_07.md) | [Part 09: Navigation & Routing →](part_09.md)
