
## When we want to use java validation library over the instance fields defined in the Request DTO

Java type	Typical annotation
String	    @NotBlank
BigDecimal	@NotNull, @DecimalMin
Integer / Long	@NotNull, @Min, @Max
List	@NotEmpty, @Size
Object	@NotNull, possibly @Valid
Enum	@NotNull