在 Mockito 中实现 `if..else` 逻辑主要通过 **参数匹配** 和 **条件模拟** 两种方式实现。以下是具体案例：

---

### 方法1：参数匹配（推荐）
根据不同的输入参数返回不同结果：
```java
import static org.mockito.Mockito.*;

// 模拟服务
UserService mockService = mock(UserService.class);

// 当输入参数为1时返回"Admin"
when(mockService.getRole(1)).thenReturn("Admin");

// 当输入参数为2时返回"User"
when(mockService.getRole(2)).thenReturn("User");

// 其他参数返回"Guest"
when(mockService.getRole(anyInt())).thenReturn("Guest"); // 注意顺序！

// 测试验证
assertEquals("Admin", mockService.getRole(1));  // 通过
assertEquals("User", mockService.getRole(2));   // 通过
assertEquals("Guest", mockService.getRole(99)); // 通过
```

> **注意**：具体匹配规则（如 `eq(1)`）需放在通用规则（如 `anyInt()`）之前，否则会被覆盖！

---

### 方法2：使用 `Answer` 实现复杂逻辑
通过回调实现动态条件判断：
```java
import org.mockito.stubbing.Answer;

when(mockService.checkAccess(anyString())).thenAnswer(new Answer<Boolean>() {
    @Override
    public Boolean answer(InvocationOnMock invocation) {
        String token = invocation.getArgument(0); // 获取参数
        // 实现if..else逻辑
        if ("admin123".equals(token)) {
            return true;
        } else if ("user456".equals(token)) {
            return false;
        } else {
            throw new IllegalArgumentException("Invalid token");
        }
    }
});

// 测试验证
assertTrue(mockService.checkAccess("admin123"));  // 通过
assertFalse(mockService.checkAccess("user456"));   // 通过
assertThrows(IllegalArgumentException.class, () -> 
    mockService.checkAccess("invalid"));           // 通过
```

**Lambda 简化版** (Java 8+)：
```java
when(mockService.checkAccess(anyString())).thenAnswer(invocation -> {
    String token = invocation.getArgument(0);
    return switch (token) {
        case "admin123" -> true;
        case "user456" -> false;
        default -> throw new IllegalArgumentException("Invalid token");
    };
});
```

---

### 关键点总结
| **方法**       | **适用场景**                     | **示例**                              |
|----------------|--------------------------------|---------------------------------------|
| 参数匹配        | 根据不同输入返回固定值           | `when(obj.method(eq(1))).thenReturn(x)` |
| `Answer` 接口  | 需要动态计算或复杂分支逻辑       | 使用 `thenAnswer()` + 条件判断         |

> 💡 **最佳实践**：  
> - 简单分支用 **参数匹配**（代码简洁）  
> - 复杂逻辑（如参数计算、异常抛出）用 **`Answer`**  
> - 始终将具体匹配规则（如 `eq()`）放在通用规则（如 `any()`）之前
