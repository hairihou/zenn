---
title: 'MUIのSelectをちょっとだけ使いやすくする'
emoji: '📝'
type: 'tech'
topics:
  - 'react'
  - 'mui'
published: true
---

<!-- easier-to-use-mui-select -->

## 概要

`@mui/material` の `Select` と `TextField` を一緒に使う場面が多かったため、 `TextField` に構造を寄せる形で使い回しが利くようにしました。

![](/images/easier-to-use-mui-select/sample-component.png)

## まずはそのまま使ってみる

勿論そのまま使っても問題なく動きますが、

`TextField` が

- `FormControl`
- `InputLabel`
- `Input`
- `FormHelperText`

から構成されているのに対し、 `Select` は labelやhelp textを持ちません。そのため `TextField` と同じように振る舞わせてあげる場合は以下のように実装する必要があります。

```tsx
import {
  Container,
  FormControl,
  FormHelperText,
  InputLabel,
  MenuItem,
  Select,
  SelectChangeEvent,
  Stack,
  TextField,
} from '@mui/material';
import { ChangeEvent, FC, useState } from 'react';

export const SampleComponent: FC = () => {
  const [name, setName] = useState('');
  const [age, setAge] = useState('');

  const [textFieldError, setTextFieldError] = useState<string>();
  const [selectError, setSelectError] = useState<string>();

  const handleTextFieldChange = (event: ChangeEvent<HTMLInputElement | HTMLTextAreaElement>): void => {
    setName(event.target.value);
  };

  const handleSelectChange = (event: SelectChangeEvent): void => {
    setAge(event.target.value as unknown as string);
  };

  return (
    <Container>
      <Stack spacing={4}>
        {/* TextFieldはこれだけでいい */}
        <TextField
          id="demo-simple-text-field-outlined"
          label="Name"
          onChange={handleTextFieldChange}
          value={name}
          error={!!textFieldError}
          helperText={textFieldError}
        ></TextField>

        {/* Selectはlabelやhelp textを自分で書く必要がある */}
        <FormControl error={!!selectError}>
          <InputLabel id="demo-simple-select-outlined-label">Age</InputLabel>
          <Select
            labelId="demo-simple-select-outlined-label"
            id="demo-simple-select-outlined"
            value={age}
            label="Age"
            onChange={handleSelectChange}
          >
            <MenuItem value="">
              <em>None</em>
            </MenuItem>
            <MenuItem value={10}>Ten</MenuItem>
            <MenuItem value={20}>Twenty</MenuItem>
            <MenuItem value={30}>Thirty</MenuItem>
          </Select>
          {!!selectError && <FormHelperText>{selectError}</FormHelperText>}
        </FormControl>
      </Stack>
    </Container>
  );
};
```

## FormControl + SelectのWrapper Componentを作る

それでは上記の実装をスッキリさせるために `FormControl` と `Select`を組み合わせたWrapper Componentを作ります。`TextField` に寄せて `SelectField` という名前にします。
共通化する上で不要になるPropsは事前に削っておきます。各プロパティの適用されるコンポーネントを明確にするためにPropsの中に `SelectProps` を作っていますが、不要であればフラットにしてしまっても大丈夫です。(プロダクトで使う場合はもっと絞られる想定)

```tsx
import { FormControl, FormControlProps, FormHelperText, InputLabel, Select, SelectProps } from '@mui/material';
import { FC, ReactNode, useMemo } from 'react';

interface Props extends Omit<FormControlProps, 'children' | 'id' | 'onChange'> {
  helperText?: ReactNode;
  label?: ReactNode;
  SelectProps?: Omit<SelectProps, 'label' | 'labelId' | 'variant'>;
}

export const SelectField: FC<Props> = ({ helperText, label, SelectProps, ...props }) => {
  const labelId = useMemo(
    () => (SelectProps?.id !== undefined ? `${SelectProps?.id}-label` : undefined),
    [SelectProps?.id]
  );

  return (
    <FormControl {...props}>
      <InputLabel id={labelId}>{label}</InputLabel>
      <Select
        {...SelectProps}
        id={SelectProps?.id}
        labelId={labelId}
        label={label}
        value={SelectProps?.value}
        onChange={SelectProps?.onChange}
      >
        {SelectProps?.children}
      </Select>
      {!!helperText && <FormHelperText>{helperText}</FormHelperText>}
    </FormControl>
  );
};
```

これにより元の実装は以下のようになります。 form control, label, help textが無くなって若干スッキリしましたね。

```tsx
import { Container, MenuItem, SelectChangeEvent, Stack, TextField } from '@mui/material';
import { ChangeEvent, FC, useState } from 'react';
import { SelectField } from '../select-field/SelectField';

export const SampleComponent: FC = () => {
  const [name, setName] = useState('');
  const [age, setAge] = useState('');

  const [textFieldError, setTextFieldError] = useState<string>();
  const [selectError, setSelectError] = useState<string>();

  const handleTextFieldChange = (event: ChangeEvent<HTMLInputElement | HTMLTextAreaElement>): void => {
    setName(event.target.value);
  };

  const handleSelectChange = (event: SelectChangeEvent<unknown>): void => {
    setAge(event.target.value as string);
  };

  return (
    <Container>
      <Stack spacing={4}>
        <TextField
          id="demo-simple-text-field-outlined"
          label="Name"
          onChange={handleTextFieldChange}
          value={name}
          error={!!textFieldError}
          helperText={textFieldError}
        ></TextField>

        <SelectField
          error={!!selectError}
          label="Age"
          helperText={selectError}
          SelectProps={{
            id: 'demo-simple-select-outlined',
            onChange: handleSelectChange,
            value: age,
            children: [
              <MenuItem key="none" value="">
                <em>None</em>
              </MenuItem>,
              <MenuItem key="ten" value={10}>
                Ten
              </MenuItem>,
              <MenuItem key="twenty" value={20}>
                Twenty
              </MenuItem>,
              <MenuItem key="thirty" value={30}>
                Thirty
              </MenuItem>,
            ],
          }}
        ></SelectField>
      </Stack>
    </Container>
  );
};
```

## 補足: forwardRefを付ける場合

`react-hook-form` 等を使う場合は子コンポーネント(今回だとselect)に参照を渡せないとエラーになってしまいます。
なので `SelectField` 内で `forwardRef` で参照を渡すようにしておきます。実装は以下になります。

```tsx
import { FormControl, FormControlProps, FormHelperText, InputLabel, Select, SelectProps } from '@mui/material';
import { FC, ReactNode, forwardRef, useMemo } from 'react';

interface Props extends Omit<FormControlProps, 'children' | 'id' | 'onChange'> {
  helperText?: ReactNode;
  label?: ReactNode;
  SelectProps?: Omit<SelectProps, 'label' | 'labelId' | 'ref' | 'variant'>;
}

const SelectField: FC<Props> = forwardRef<HTMLDivElement, Props>(
  ({ helperText, label, SelectProps, ...props }, ref) => {
    const labelId = useMemo(
      () => (SelectProps?.id !== undefined ? `${SelectProps?.id}-label` : undefined),
      [SelectProps?.id]
    );

    return (
      <FormControl {...props}>
        <InputLabel id={labelId}>{label}</InputLabel>
        <Select
          {...SelectProps}
          id={SelectProps?.id}
          labelId={labelId}
          label={label}
          value={SelectProps?.value}
          onChange={SelectProps?.onChange}
          ref={ref}
        >
          {SelectProps?.children}
        </Select>
        {!!helperText && <FormHelperText>{helperText}</FormHelperText>}
      </FormControl>
    );
  }
);

SelectField.displayName = 'SelectField';

export { SelectField };
```

## まとめ

これで `TextField` と同じ感覚で `Select` を使えるようになりました。MUIはカスタマイズ無しでもかなり使い勝手の良いUIフレームワークなので今後も使い倒していきます。
