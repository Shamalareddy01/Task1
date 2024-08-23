# Task1
public class Supplier {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long supplierId;

    private String companyName;
    private String website;
    private String location;
    private String natureOfBusiness;
    private String manufacturingProcesses;

    // Getters and setters
}
public interface SupplierRepository extends JpaRepository<Supplier, Long> {
    Page<Supplier> findByLocationAndNatureOfBusinessAndManufacturingProcesses(
        String location, String natureOfBusiness, String manufacturingProcesses, Pageable pageable);
}
@Service
public class SupplierService {
    @Autowired
    private SupplierRepository supplierRepository;

    public Page<Supplier> searchSuppliers(String location, String natureOfBusiness, String manufacturingProcesses, int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return supplierRepository.findByLocationAndNatureOfBusinessAndManufacturingProcesses(location, natureOfBusiness, manufacturingProcesses, pageable);
    }
}
@RestController
@RequestMapping("/api/supplier")
public class SupplierController {
    @Autowired
    private SupplierService supplierService;

    @PostMapping("/query")
    public ResponseEntity<Page<Supplier>> querySuppliers(
        @RequestParam String location,
        @RequestParam String natureOfBusiness,
        @RequestParam String manufacturingProcesses,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
        Page<Supplier> suppliers = supplierService.searchSuppliers(location, natureOfBusiness, manufacturingProcesses, page, size);
        return ResponseEntity.ok(suppliers);
    }
}
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
}
@ApiOperation(value = "Search suppliers based on criteria")
@PostMapping("/query")
public ResponseEntity<Page<Supplier>> querySuppliers(
    @ApiParam(value = "Location of the supplier", required = true) @RequestParam String location,
    @ApiParam(value = "Nature of the business", required = true) @RequestParam String natureOfBusiness,
    @ApiParam(value = "Manufacturing process capability", required = true) @RequestParam String manufacturingProcesses,
    @ApiParam(value = "Page number for pagination", defaultValue = "0") @RequestParam int page,
    @ApiParam(value = "Number of records per page", defaultValue = "10") @RequestParam int size) {
    ...
}
