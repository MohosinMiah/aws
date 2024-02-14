
# KMS Encrypt and Decrypt

```
import com.amazonaws.services.kms.AWSKMS;
import com.amazonaws.services.kms.AWSKMSClientBuilder;
import com.amazonaws.services.kms.model.DecryptRequest;
import com.amazonaws.services.kms.model.DecryptResult;
import com.amazonaws.services.kms.model.EncryptRequest;
import com.amazonaws.services.kms.model.EncryptResult;
import org.springframework.stereotype.Service;
import java.nio.ByteBuffer;
import java.util.Base64;

@Service
public class KMSService {

    private final AWSKMS kmsClient = AWSKMSClientBuilder.defaultClient();
    private final String kmsKeyId = "your_kms_key_id"; // Replace with your KMS key ID

    public String encryptData(String plaintext) {
        ByteBuffer plaintextBuffer = ByteBuffer.wrap(plaintext.getBytes());
        EncryptRequest encryptRequest = new EncryptRequest()
                .withKeyId(kmsKeyId)
                .withPlaintext(plaintextBuffer);
        EncryptResult encryptResult = kmsClient.encrypt(encryptRequest);
        return Base64.getEncoder().encodeToString(encryptResult.getCiphertextBlob().array());
    }

    public String decryptData(String encryptedData) {
        ByteBuffer ciphertextBuffer = ByteBuffer.wrap(Base64.getDecoder().decode(encryptedData));
        DecryptRequest decryptRequest = new DecryptRequest()
                .withCiphertextBlob(ciphertextBuffer);
        DecryptResult decryptResult = kmsClient.decrypt(decryptRequest);
        return new String(decryptResult.getPlaintext().array());
    }
}
```
# Employee CRUD Example
```
// Employee.java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // Encrypted sensitive fields
    private String encryptedSalary;

    // Getters and setters
}

// EmployeeRepository.java
import org.springframework.data.jpa.repository.JpaRepository;

public interface EmployeeRepository extends JpaRepository<Employee, Long> {
}

// EmployeeService.java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Autowired
    private KMSService kmsService;

    public Employee createEmployee(Employee employee) {
        // Encrypt sensitive data before saving
        employee.setEncryptedSalary(kmsService.encryptData(employee.getSalary()));
        return employeeRepository.save(employee);
    }

    public Employee getEmployee(Long id) {
        Employee employee = employeeRepository.findById(id).orElse(null);
        if (employee != null) {
            // Decrypt sensitive data before returning
            employee.setSalary(kmsService.decryptData(employee.getEncryptedSalary()));
        }
        return employee;
    }

    // Implement update and delete methods similarly
}

// EmployeeController.java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;

    @PostMapping
    public Employee createEmployee(@RequestBody Employee employee) {
        return employeeService.createEmployee(employee);
    }

    @GetMapping("/{id}")
    public Employee getEmployee(@PathVariable Long id) {
        return employeeService.getEmployee(id);
    }

    // Implement update and delete endpoints similarly
}
```
