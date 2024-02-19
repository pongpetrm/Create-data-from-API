# Create-data-from-API
# Answer code

```typescript
import { useEffect, useState } from "react";
import axios from "axios";
interface DataUser {
  gender: string,
  age: number,
  hair: {
    color: string
  },
  firstName: string,
  lastName: string,
  address: {
    postalCode: string
  },
  company: {
    department: string
  }
}

interface DataSummary  {
  male: number;
  female: number;
  ageRange: string;
  hair: Record<string, number>;
  addressUser: Record<string, string>;
}

async function UserSummary(data : DataUser[]) {
  const grouped: Record<string, DataSummary> = {};

  data.forEach(user => {
    const department = user?.company?.department;

    if (!grouped[department]) {
      grouped[department] = {
        male: 0,
        female: 0,
        ageRange: "",
        hair: {},
        addressUser: {}
      };
    }

    if (user.gender === 'male') {
      grouped[department].male++;
    } else {
      grouped[department].female++;
    }

    const age = user.age;
    if (!grouped[department].ageRange) {
      grouped[department].ageRange = `${age}-${age}`;
    } else {
      const [minAge, maxAge] = grouped[department].ageRange.split('-').map(Number);
      grouped[department].ageRange = `${Math.min(minAge, age)}-${Math.max(maxAge, age)}`;
    }

    const hairColor = user.hair.color;
    if (!grouped[department].hair[hairColor]) {
      grouped[department].hair[hairColor] = 0;
    }
    grouped[department].hair[hairColor]++;

    grouped[department].addressUser[`${user.firstName}${user.lastName}`] = user.address.postalCode;
  })

  return grouped
}

function App() {
  const [dataInfo, setDataInfo] = useState<Record<string, DataSummary>>({})

  useEffect(() => {
    const fetchTodos = async () => {
      try {
        const response = await axios.get<any>('https://dummyjson.com/users');
        const UserSummaryData = await UserSummary(response?.data?.users)
        setDataInfo(UserSummaryData)
      } catch (error) {
        console.error('Error fetching todos:', error);
      }
    };

    fetchTodos();
  }, []);

  return (
    <div className="App" style={ { padding: 20 } }>
      {
        Object.keys(dataInfo).map((key, index) =>
          <div key={index} style={ { padding: 20, border: '2px solid #eee', borderRadius: 10, marginBottom: 15 } }>
            
            <div>
            {index+1}) Department: {key}
            </div>
            <div>
              male: {dataInfo[key]?.male}
            </div>
            <div>
              female: {dataInfo[key]?.female}
            </div>
            <div>
              ageRange: {dataInfo[key]?.ageRange}
            </div>
            <div style={{marginTop: 10}}>
              hair:
            </div>
            <div style={ { padding: 10, border: '2px solid #eee', borderRadius: 5 } }>
              {
                Object.keys(dataInfo[key]?.hair).map((keyHair, indexHair) =>
                  <div key={indexHair}>
                    {keyHair}: {dataInfo[key]?.hair[keyHair]}
                  </div>
                )
              }
            </div>
            <div style={{marginTop: 10}}>
              addressUser:
            </div>
            <div style={ { padding: 10, border: '2px solid #eee', borderRadius: 5 } }>
              {
                Object.keys(dataInfo[key]?.addressUser).map((keyAddressUser, indexAddressUser) =>
                  <div key={indexAddressUser}>
                    {keyAddressUser}: {dataInfo[key]?.addressUser[keyAddressUser]}
                  </div>
                )
              }
            </div>
          </div>
        )
      }
    </div>
  );
}

export default App;
