
## List Tests

- create `src/__tests__/List.test.tsx` file

```tsx
import { render, screen } from '@testing-library/react';
import { describe, test, expect, vi } from 'vitest';
import List from '../components/List';
import { type Item } from '../utils';
// Mock the ItemCard component to simplify testing
// This replaces the actual ItemCard component with a simple article element
// that contains the text "item card"
vi.mock('../components/ItemCard', () => ({
  // The default export is a function that returns a simple article element
  // This helps us test the List component in isolation without the complexity
  // of the actual ItemCard implementation
  default: () => <article>item card</article>,
}));

describe('List', () => {
  const mockItems: Item[] = [
    {
      id: '1',
      title: 'Test Item 1',
      description: 'Content 1',
      category: 'urgent',
    },
    {
      id: '2',
      title: 'Test Item 2',
      description: 'Content 2',
      category: 'normal',
    },
  ];
  const mockOnDelete = vi.fn();

  test('renders the Flow Board heading', () => {
    render(<List items={mockItems} onDelete={mockOnDelete} />);
    expect(
      screen.getByRole('heading', { level: 2, name: 'Flow Board' })
    ).toBeInTheDocument();
  });

  test('renders correct number of ItemCards', () => {
    render(<List items={mockItems} onDelete={mockOnDelete} />);

    const cards = screen.getAllByRole('article');
    expect(cards).toHaveLength(2);
  });
  //
  test('renders empty grid when no items provided', () => {
    render(<List items={[]} onDelete={mockOnDelete} />);
    expect(screen.queryAllByRole('article')).toHaveLength(0);
  });
  // ALTERNATIVE if we want to test the number of items only in the component. Useful if there are other places where such items are rendered.

  test('ALTERNATIVE: renders correct number of ItemCards', () => {
    const { queryAllByRole } = render(
      <List items={mockItems} onDelete={mockOnDelete} />
    );
    expect(queryAllByRole('article')).toHaveLength(2);
  });

  test('ALTERNATIVE: renders empty grid when no items provided', () => {
    const { queryAllByRole } = render(
      <List items={[]} onDelete={mockOnDelete} />
    );
    expect(queryAllByRole('article')).toHaveLength(0);
  });
});
```

src/components/List.tsx

```tsx
import ItemCard from './ItemCard';
import { type Item } from '../utils';

export default function List({
  items,
  onDelete,
}: {
  items: Item[];
  onDelete: (id: string) => void;
}) {
  return (
    <section className='mt-8'>
      <h2 className='text-xl font-semibold mb-2'>Flow Board</h2>
      <div className='grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4'>
        {items.map((item) => (
          <ItemCard key={item.id} {...item} onDelete={onDelete} />
        ))}
      </div>
    </section>
  );
}
```

## ItemCard Tests

components/ItemCard.tsx

```tsx
import { Trash2 } from 'lucide-react';

type ItemCardProps = {
  id: string;
  title: string;
  description: string;
  category: string;
  onDelete: (id: string) => void;
};

const ItemCard = ({
  id,
  title,
  description,
  category,
  onDelete,
}: ItemCardProps) => {
  return <div>ItemCard</div>;
};
export default ItemCard;
```

- create `src/__tests__/ItemCard.test.tsx` file

```tsx
import { describe, test, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import ItemCard from '../components/ItemCard';
import { type Item } from '../utils';

type MockProps = Item & { onDelete: () => void };

describe('ItemCard', () => {
  const mockProps: MockProps = {
    id: '1',
    title: 'Test Task',
    description: 'Test Description',
    category: 'urgent',
    onDelete: vi.fn(),
  };

  test('renders card with correct content', () => {
    render(<ItemCard {...mockProps} />);

    expect(
      screen.getByRole('heading', { name: 'Test Task' })
    ).toBeInTheDocument();
    expect(screen.getByRole('article')).toBeInTheDocument();
    expect(screen.getByText('Test Description')).toBeInTheDocument();
    expect(screen.getByText('urgent')).toBeInTheDocument();
  });

  test('calls onDelete when delete button is clicked', async () => {
    const user = userEvent.setup();
    render(<ItemCard {...mockProps} />);

    const deleteButton = screen.getByRole('button', {
      name: 'Delete task: 1',
    });
    await user.click(deleteButton);

    expect(mockProps.onDelete).toHaveBeenCalledWith('1');
  });
});
```

## Context API

You can use current to setup to setup integration tests for App.tsx but in my case I will show you how to test components that use the context.

- create `src/AppWithContext.tsx` file

```tsx
const AppWithContext = () => {
  return <div>AppWithContext</div>;
};
export default AppWithContext;
```

- create `src/FlowContext.tsx` file

```tsx
import { createContext, useContext, ReactNode, useState } from 'react';
import { Item, ItemWithoutId } from './utils';

interface FlowContextType {
  items: Item[];
  handleAddItem: (newItem: ItemWithoutId) => void;
  handleDeleteItem: (id: string) => void;
}

const FlowContext = createContext<FlowContextType | undefined>(undefined);

export function FlowProvider({ children }: { children: ReactNode }) {
  const [items, setItems] = useState<Item[]>([]);

  const handleAddItem = (newItem: ItemWithoutId) => {
    setItems([...items, { ...newItem, id: Date.now().toString() }]);
  };

  const handleDeleteItem = (id: string) => {
    setItems((prev) => prev.filter((item) => item.id !== id));
  };

  return (
    <FlowContext.Provider value={{ items, handleAddItem, handleDeleteItem }}>
      {children}
    </FlowContext.Provider>
  );
}

export function useFlowContext() {
  const context = useContext(FlowContext);
  if (context === undefined) {
    throw new Error('useFlow must be used within a FlowProvider');
  }
  return context;
}
```

src/main.tsx

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
// import App from './App.tsx';
import AppWithContext from './AppWithContext.tsx';
import './index.css';
import { FlowProvider } from './FlowContext.tsx';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <FlowProvider>
      <AppWithContext />
    </FlowProvider>
  </StrictMode>
);
```

src/AppWithContext.tsx

```tsx
import Form from './components/Form';
import List from './components/List';
import { useFlowContext } from './FlowContext';
const AppWithContext = () => {
  const { items, handleAddItem, handleDeleteItem } = useFlowContext();
  return (
    <main className='container mx-auto p-4 max-w-6xl'>
      <h1 className='text-3xl font-bold mb-8'>Focus Flow</h1>
      <Form onSubmit={handleAddItem} />
      <List items={items} onDelete={handleDeleteItem} />
    </main>
  );
};
export default AppWithContext;
```

## AppWithContext Tests

- create `src/__tests__/AppWithContext.test.tsx` file

```tsx
import { render, screen } from '@testing-library/react';
import userEvent, { UserEvent } from '@testing-library/user-event';
import { describe, test, expect, beforeEach, vi } from 'vitest';
import { FlowProvider } from '../FlowContext';
import AppWithContext from '../AppWithContext';

const getElements = () => ({
  titleInput: screen.getByRole('textbox', { name: /title/i }),
  descriptionInput: screen.getByRole('textbox', { name: /description/i }),
  categorySelect: screen.getByRole('combobox', { name: /category/i }),
  submitButton: screen.getByRole('button', { name: /add task/i }),
});

const customRenderAppWithContext = () => {
  return render(
    <FlowProvider>
      <AppWithContext />
    </FlowProvider>
  );
};

const addTestItem = async (user: UserEvent) => {
  const { titleInput, descriptionInput, categorySelect, submitButton } =
    getElements();
  await user.type(titleInput, 'Test Item');
  await user.type(descriptionInput, 'Test Content');
  await user.selectOptions(categorySelect, 'urgent');
  await user.click(submitButton);
};

describe('AppWithContext', () => {
  let user: UserEvent;

  beforeEach(() => {
    vi.clearAllMocks();
    user = userEvent.setup();
    customRenderAppWithContext();
  });

  test('renders heading and form elements', () => {
    expect(
      screen.getByRole('heading', { level: 1, name: 'Focus Flow' })
    ).toBeInTheDocument();
    // Verify all form elements are present
    const elements = getElements();
    Object.values(elements).forEach((element) => {
      expect(element).toBeInTheDocument();
    });
  });

  test('handles adding an item', async () => {
    const cardsBefore = screen.queryAllByRole('article');
    expect(cardsBefore).toHaveLength(0);

    await addTestItem(user);

    // Use getByText instead of findByText since we already have findAllByRole above
    const cardsAfter = await screen.findAllByRole('article');
    expect(cardsAfter).toHaveLength(1);
    expect(screen.getByText('Test Item')).toBeInTheDocument();
    expect(screen.getByText('Test Content')).toBeInTheDocument();
    expect(screen.getByText('urgent')).toBeInTheDocument();
  });

  test('handles deleting an item', async () => {
    await addTestItem(user);

    const deleteButton = screen.getByRole('button', { name: /delete/i });
    expect(deleteButton).toBeInTheDocument(); // Verify delete button exists

    await user.click(deleteButton);
    expect(screen.queryByText('Test Item')).not.toBeInTheDocument(); // Verify item content is gone
    expect(screen.queryAllByRole('article')).toHaveLength(0);
  });
});
```