Here is a reproducer to show what kind of issues can happen when casting a MoneyType field value back to an int for large numbers.

To see it in action, clone the repository, then:  
`symfony serve -d`  
then navigate to /article, and try inserting a large number (still < PHP_INT_MAX i guess) to see the differences happening

For instance, when inputing 111111111111111110,00, i get the following results 

![image](https://github.com/JacquesDurand/money_type_reproducer/assets/59364973/fb65a312-a49d-4069-abbf-b8ed41af7c59)

When dumping in `Symfony\Component\Form\Extension\Core\DataTransformer\MoneyToLocalizedStringTransformer` the following way: 

```php
    public function reverseTransform(mixed $value): int|float|null
    {
        $value = parent::reverseTransform($value);
        if (null !== $value && 1 !== $this->divisor) {
            $value = (string) ($value * $this->divisor);
            dd("string value: ".$value, "int value: ".(int) $value, "float value: " .(float)$value);
            $value = 'integer' === $this->modelType ? (int) $value : (float) $value;
        }

        return $value;
    }
```

The final int value is not the same as the one from the input.
