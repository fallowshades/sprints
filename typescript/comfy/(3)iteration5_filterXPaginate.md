## Install Shadcn Form Components

- [Form Component](https://ui.shadcn.com/docs/components/form)

- Label, Input, Select, Slider, Checkbox

```sh
npx shadcn-ui@latest add label input select slider checkbox
```

## Filters - Initial Setup

```tsx
import { Form, useLoaderData, Link } from "react-router-dom";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Button } from "./ui/button";

function Filters() {
  return (
    <Form className="border rounded-md px-8 py-4 grid gap-x-4 gap-y-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 items-center">
      <div className="mb-2">
        <Label htmlFor="search">Search Product</Label>
        <Input id="search" name="search" type="text" defaultValue="" />
      </div>
      <Button type="submit" size="sm" className="self-end mb-2">
        search
      </Button>
      <Button
        type="button"
        asChild
        size="sm"
        variant="outline"
        className="self-end mb-2"
      >
        <Link to="/products">reset</Link>
      </Button>
    </Form>
  );
}
export default Filters;
```

## API

- [API DOCS](https://documenter.getpostman.com/view/18152321/2s9Xy5KpTi)

## Products Loader

```ts
export const loader: LoaderFunction = async ({
  request,
}): Promise<ProductsResponse> => {
  const params = Object.fromEntries([
    ...new URL(request.url).searchParams.entries(),
  ]);

  const response = await customFetch<ProductsResponse>(url, { params });
  console.log(response.data);

  return { ...response.data, params };
};
```

new URL(request.url) creates a new URL object from the URL in the request.
.searchParams.entries() gets an iterator for entries in the query parameters, where each entry is an array of [key, value].

... is the spread operator, which expands the entries into individual elements.
Object.fromEntries([...]) converts these entries back into an object, where each key-value pair becomes a property in the object.

So, if your URL is http://example.com?param1=value1&param2=value2, the resulting params object would be { param1: 'value1', param2: 'value2' }.

## Setup Params Type

utils/types.ts

```ts
export type Params = {
  search?: string;
  category?: string;
  company?: string;
  order?: string;
  price?: string;
  shipping?: string;
  page?: number;
};

export type ProductsResponseWithParams = ProductsResponse & { params: Params };
```

## Implement Params

- in Products setup loader return : Response<ProductsResponseWithParams>

Filters.tsx

```tsx
import { type ProductsResponseWithParams } from "@/utils";
function Filters() {
  const { meta, params } = useLoaderData() as ProductsResponseWithParams;
  const { search } = params;
  return (
    <Form>
      <div>
        <Label htmlFor="search">Search Product</Label>
        <Input id="search" name="search" type="text" defaultValue={search} />
      </div>
    </Form>
  );
}
export default Filters;
```

## FormInput Component

- create components/FormInput.tsx
- import and setup in Filters

```tsx
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

type FormInputProps = {
  name: string;
  type: string;
  label?: string;
  defaultValue?: string;
};

function FormInput({ label, name, type, defaultValue }: FormInputProps) {
  return (
    <div className="mb-2">
      <Label htmlFor={name} className="capitalize">
        {label || name}
      </Label>
      <Input id={name} name={name} type={type} defaultValue={defaultValue} />
    </div>
  );
}
export default FormInput;
```

```tsx
import FormInput from "./FormInput";

function Filters() {
  const { meta, params } = useLoaderData() as ProductsResponseWithParams;
  const { search, company, category, shipping, order, price } = params;

  return (
    <Form className="border rounded-md px-8 py-4 grid gap-x-4 gap-y-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 items-center">
      {/* search */}
      <FormInput
        type="search"
        label="search product"
        name="search"
        defaultValue={search}
      />
    </Form>
  );
}
```

## FormSelect.tsx

[Shadcn Select](https://ui.shadcn.com/docs/components/select)

- create components/FormSelect.tsx
- render in Filters.tsx

```tsx
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";

import { Label } from "@/components/ui/label";

type SelectInputProps = {
  name: string;
  label?: string;
  defaultValue?: string;
  options: string[];
};

function SelectInput({ label, name, options, defaultValue }: SelectInputProps) {
  return (
    <div className="mb-2">
      <Label htmlFor={name} className="capitalize">
        {label || name}
      </Label>
      <Select defaultValue={defaultValue || options[0]} name={name}>
        <SelectTrigger id={name}>
          <SelectValue />
        </SelectTrigger>
        <SelectContent>
          {options.map((item) => {
            return (
              <SelectItem key={item} value={item}>
                {item}
              </SelectItem>
            );
          })}
        </SelectContent>
      </Select>
    </div>
  );
}
export default SelectInput;
```

Filters.tsx

```tsx
{
  /* CATEGORIES */
}
<FormSelect
  label="select category"
  name="category"
  options={meta.categories}
  defaultValue={category}
/>;
{
  /* COMPANIES */
}
<FormSelect
  label="select company"
  name="company"
  options={meta.companies}
  defaultValue={company}
/>;
{
  /* ORDER */
}

<FormSelect
  label="order by"
  name="order"
  options={["a-z", "z-a", "high", "low"]}
  defaultValue={order}
/>;
```

## FormRange

[Shadcn Slider](https://ui.shadcn.com/docs/components/slider)

Filters.tsx

```tsx
<FormRange label="price" name="price" defaultValue={price} />
```

FormRange.tsx

```tsx
import { formatAsDollars } from "@/utils";
import { useState } from "react";

import { Label } from "@/components/ui/label";
import { Slider } from "./ui/slider";
type FormRangeProps = {
  name: string;
  label?: string;
  defaultValue?: string;
};

function FormRange({ name, label, defaultValue }: FormRangeProps) {
  const step = 1000;
  const maxPrice = 100000;
  const defaultPrice = defaultValue ? Number(defaultValue) : maxPrice;
  const [selectedPrice, setSelectedPrice] = useState(defaultPrice);

  return (
    <div className="mb-2">
      <Label htmlFor={name} className="capitalize flex justify-between">
        {label || name}
        <span>{formatAsDollars(selectedPrice)}</span>
      </Label>
      <Slider
        id={name}
        name={name}
        step={step}
        max={maxPrice}
        value={[selectedPrice]}
        onValueChange={(value) => setSelectedPrice(value[0])}
        className="mt-4"
      />
    </div>
  );
}
export default FormRange;
```

## FormCheckbox

[Shadcn Checkbox](https://ui.shadcn.com/docs/components/checkbox)

Filters.tsx

```tsx
<FormCheckbox label="free shipping" name="shipping" defaultValue={shipping} />
```

FormCheckbox.tsx

```tsx
import { Label } from "@/components/ui/label";
import { Checkbox } from "@/components/ui/checkbox";

type FormCheckboxProps = {
  name: string;
  label?: string;
  defaultValue?: string;
};

function FormCheckbox({ name, label, defaultValue }: FormCheckboxProps) {
  const defaultChecked = defaultValue === "on" ? true : false;

  return (
    <div className="mb-2 flex justify-between self-end">
      <Label htmlFor={name} className="capitalize">
        {label || name}
      </Label>
      <Checkbox id={name} name={name} defaultChecked={defaultChecked} />
    </div>
  );
}
export default FormCheckbox;
```

## Pagination

[Shadcn Pagination](https://ui.shadcn.com/docs/components/pagination)

```sh
npx shadcn-ui@latest add pagination

```

- customize

pagination.tsx

```tsx
import { Link } from "react-router-dom";

type PaginationLinkProps = {
  isActive?: boolean;
} & Pick<ButtonProps, "size"> &
  // Link
  React.ComponentProps<typeof Link>;

const PaginationLink = ({
  className,
  isActive,
  size = "icon",
  ...props
}: PaginationLinkProps) => (
  // Link
  <Link
    aria-current={isActive ? "page" : undefined}
    className={cn(
      buttonVariants({
        variant: isActive ? "outline" : "ghost",
        size,
      }),
      className
    )}
    {...props}
  />
);
```

## Pagination - Setup

- create utils/pagination.ts
- setup export

```ts
export * from "./pagination";
```

```ts
type ConstructUrlParams = {
  pageNumber: number;
  search: string;
  pathname: string;
};

export const constructUrl = ({
  pageNumber,
  search,
  pathname,
}: ConstructUrlParams) => {
  return `/products`;
};

type ConstructPrevOrNextParams = {
  currentPage: number;
  pageCount: number;
  search: string;
  pathname: string;
};

export const constructPrevOrNextUrl = ({
  currentPage,
  pageCount,
  search,
  pathname,
}: ConstructPrevOrNextParams): { prevUrl: string; nextUrl: string } => {
  const prevUrl = "/products";
  const nextUrl = "/products";
  return { prevUrl, nextUrl };
};
```

## Pagination Container

```tsx
import {
  Pagination,
  PaginationContent,
  PaginationItem,
  PaginationLink,
  PaginationNext,
  PaginationPrevious,
} from "@/components/ui/pagination";
import {
  ProductsResponseWithParams,
  constructUrl,
  constructPrevOrNextUrl,
} from "@/utils";

import { useLoaderData, useLocation } from "react-router-dom";

function PaginationContainer() {
  const { meta } = useLoaderData() as ProductsResponseWithParams;
  const { pageCount, page } = meta.pagination;

  const { search, pathname } = useLocation();

  const pages = Array.from({ length: pageCount }, (_, index) => index + 1);

  if (pageCount < 2) return null;

  const renderPagination = pages.map((pageNumber) => {
    const isActive = pageNumber === page;
    const url = constructUrl({ pageNumber, search, pathname });

    return (
      <PaginationItem key={pageNumber}>
        <PaginationLink to={url} isActive={isActive}>
          {pageNumber}
        </PaginationLink>
      </PaginationItem>
    );
  });
  const { prevUrl, nextUrl } = constructPrevOrNextUrl({
    currentPage: page,
    pageCount,
    search,
    pathname,
  });

  return (
    <Pagination className="mt-16">
      <PaginationContent>
        <PaginationItem>
          <PaginationPrevious to={prevUrl} />
        </PaginationItem>
        {renderPagination}
        <PaginationItem>
          <PaginationNext to={nextUrl} />
        </PaginationItem>
      </PaginationContent>
    </Pagination>
  );
}
export default PaginationContainer;
```

## Pagination - Complete

```tsx
type ConstructUrlParams = {
  pageNumber: number;
  search: string;
  pathname: string;
};

export const constructUrl = ({
  pageNumber,
  search,
  pathname,
}: ConstructUrlParams) => {
  const searchParams = new URLSearchParams(search);
  searchParams.set("page", pageNumber.toString());
  return `${pathname}?${searchParams.toString()}`;
};

type ConstructPrevOrNextParams = {
  currentPage: number;
  pageCount: number;
  search: string;
  pathname: string;
};

export const constructPrevOrNextUrl = ({
  currentPage,
  pageCount,
  search,
  pathname,
}: ConstructPrevOrNextParams): { prevUrl: string; nextUrl: string } => {
  let prevPage = currentPage - 1;
  if (prevPage < 1) prevPage = pageCount;
  const prevUrl = constructUrl({ pageNumber: prevPage, search, pathname });

  let nextPage = currentPage + 1;
  if (nextPage > pageCount) nextPage = 1;
  const nextUrl = constructUrl({ pageNumber: nextPage, search, pathname });
  return { prevUrl, nextUrl };
};
```

## ComplexPaginationContainer

```tsx
import {
  Pagination,
  PaginationContent,
  PaginationItem,
  PaginationLink,
  PaginationNext,
  PaginationPrevious,
  PaginationEllipsis,
} from "@/components/ui/pagination";
import {
  constructUrl,
  constructPrevOrNextUrl,
  type ProductsResponseWithParams,
} from "@/utils";

import { useLoaderData, useLocation } from "react-router-dom";

function ComplexPaginationContainer() {
  const { meta } = useLoaderData() as ProductsResponseWithParams;
  const { pageCount, page } = meta.pagination;

  const { search, pathname } = useLocation();

  if (pageCount < 2) return null;

  // const renderPagination = pages.map((pageNumber) => {
  //   const isActive = pageNumber === page;
  //   const url = constructUrl(pageNumber, search, pathname);

  //   return (
  //     <PaginationItem key={pageNumber}>
  //       <PaginationLink to={url} isActive={isActive}>
  //         {pageNumber}
  //       </PaginationLink>
  //     </PaginationItem>
  //   );
  // });

  const constructButton = ({
    pageNumber,
    isActive,
  }: {
    pageNumber: number;
    isActive: boolean;
  }): React.ReactNode => {
    const url = constructUrl({ pageNumber, search, pathname });
    return (
      <PaginationItem key={pageNumber}>
        <PaginationLink to={url} isActive={isActive}>
          {pageNumber}
        </PaginationLink>
      </PaginationItem>
    );
  };

  const constructEllipsis = (key: string): React.ReactNode => {
    return (
      <PaginationItem key={key}>
        <PaginationEllipsis />
      </PaginationItem>
    );
  };

  const renderPagination = () => {
    let pages: React.ReactNode[] = [];
    // first page
    pages.push(constructButton({ pageNumber: 1, isActive: page === 1 }));
    // ellipsis
    if (page > 2) {
      pages.push(constructEllipsis("dots-1"));
    }
    // active page
    if (page !== 1 && page !== pageCount) {
      pages.push(constructButton({ pageNumber: page, isActive: true }));
    }
    // ellipsis
    if (page < pageCount - 1) {
      pages.push(constructEllipsis("dots-2"));
    }
    // last page
    pages.push(
      constructButton({ pageNumber: pageCount, isActive: page === pageCount })
    );
    return pages;
  };
  const { prevUrl, nextUrl } = constructPrevOrNextUrl({
    currentPage: page,
    pageCount,
    search,
    pathname,
  });
  return (
    <Pagination className="mt-16">
      <PaginationContent>
        <PaginationItem>
          <PaginationPrevious to={prevUrl} />
        </PaginationItem>
        {renderPagination()}
        <PaginationItem>
          <PaginationNext to={nextUrl} />
        </PaginationItem>
      </PaginationContent>
    </Pagination>
  );
}
export default ComplexPaginationContainer;
```

## SingleProduct Type

- utils/types.ts

```ts
export type SingleProductResponse = {
  data: Product;
  meta: {};
};
```

## SingleProduct - Params

- import and setup loader in App.tsx

```tsx
import { useLoaderData } from "react-router-dom";
import { Link } from "react-router-dom";
import {
  customFetch,
  formatAsDollars,
  type SingleProductResponse,
} from "@/utils";
import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Separator } from "@/components/ui/separator";

import { type LoaderFunction } from "react-router-dom";

export const loader: LoaderFunction = async ({ params }) => {
  console.log(params);
  return null;
};

const SingleProduct = () => {
  return <h1 className="text-4xl">SingleProduct Page</h1>;
};
export default SingleProduct;
```

## SingleProduct - Load Products

```tsx
export const loader: LoaderFunction = async ({
  params,
}): Promise<SingleProductResponse> => {
  const response = await customFetch<SingleProductResponse>(
    `/products/${params.id}`
  );

  return { ...response.data };
};

const SingleProduct = () => {
  const { data: product } = useLoaderData() as SingleProductResponse;
  const { image, title, price, description, colors, company } =
    product.attributes;
  const dollarsAmount = formatAsDollars(price);
  const [productColor, setProductColor] = useState(colors[0]);
  const [amount, setAmount] = useState(1);

  const addToCart = () => {
    consol.log("add to cart");
  };

  return <h1 className="text-4xl">SingleProduct Page</h1>;
};
export default SingleProduct;
```

## SingleProduct - Initial Return

```tsx
<section>
  <div className="flex gap-x-2 h-6 items-center">
    <Button asChild variant="link" size="sm">
      <Link to="/">Home</Link>
    </Button>
    <Separator orientation="vertical" />
    <Button asChild variant="link" size="sm">
      <Link to="/products">Products</Link>
    </Button>
  </div>
  {/* PRODUCT */}
  <div className="mt-6 grid gap-y-8 lg:grid-cols-2  lg:gap-x-16">
    {/* IMAGE */}
    <img
      src={image}
      alt={title}
      className="w-96 h-96 object-cover rounded-lg lg:w-full"
    />
    {/* PRODUCT INFO */}
    <div>
      <h1 className="capitalize text-3xl font-bold">{title}</h1>
      <h4 className="text-xl mt-2">{company}</h4>
      <p className="mt-3 text-md bg-muted inline-block p-2 rounded-md">
        {dollarsAmount}
      </p>
      <p className="mt-6 leading-8">{description}</p>
      {/* COLORS */}

      {/* AMOUNT */}

      {/* CART BUTTON */}
      <Button size="lg" className="mt-10" onClick={addToCart}>
        Add to bag
      </Button>
    </div>
  </div>
</section>
```

## SelectProductColor

- import/export

```tsx
type SelectProductColorProps = {
  colors: string[];
  productColor: string;
  setProductColor: React.Dispatch<React.SetStateAction<string>>;
};

function SelectProductColor({
  colors,
  productColor,
  setProductColor,
}: SelectProductColorProps) {
  return (
    <div className="mt-6">
      <h4 className="text-md font-medium tracking-wider capitalize">colors</h4>

      <div className="mt-2">
        {colors.map((color) => {
          return (
            <button
              key={color}
              type="button"
              className={`rounded-full w-6 h-6 mr-2 border-2  ${
                color === productColor && " border-primary"
              }`}
              style={{ backgroundColor: color }}
              onClick={() => setProductColor(color)}
            ></button>
          );
        })}
      </div>
    </div>
  );
}
export default SelectProductColor;
```

SingleProduct.tsx

```tsx
<SelectProductColor
  colors={colors}
  productColor={productColor}
  setProductColor={setProductColor}
/>
```
