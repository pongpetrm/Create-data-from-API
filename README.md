# Create-data-from-API
# Answer code

```typescript
import { useEffect, useState } from "react";
import axios from "axios";

// Define interfaces
interface UserData {
  gender: string;
  age: number;
  hair: {
    color: string;
  };
  firstName: string;
  lastName: string;
  address: {
    postalCode: string;
  };
  company: {
    department: string;
  };
}

interface DataSummary {
  male: number;
  female: number;
  ageRange: string;
  hair: Record<string, number>;
  addressUser: Record<string, string>;
}

// Function to compute user summary
async function computeUserSummary(data: UserData[]): Promise<Record<string, DataSummary>> {
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

    // Increment male/female count
    user.gender === 'male' ? grouped[department].male++ : grouped[department].female++;

    // Update age range
    const age = user.age;
    if (!grouped[department].ageRange) {
      grouped[department].ageRange = `${age}-${age}`;
    } else {
      const [minAge, maxAge] = grouped[department].ageRange.split('-').map(Number);
      grouped[department].ageRange = `${Math.min(minAge, age)}-${Math.max(maxAge, age)}`;
    }

    // Update hair color count
    const hairColor = user.hair.color;
    grouped[department].hair[hairColor] = (grouped[department].hair[hairColor] || 0) + 1;

    // Map user to address
    grouped[department].addressUser[`${user.firstName}${user.lastName}`] = user.address.postalCode;
  });

  return grouped;
}

export default function UserSummaryApp() {
  const [dataInfo, setDataInfo] = useState<Record<string, DataSummary>>({});

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await axios.get<any>('https://dummyjson.com/users');
        const userSummaryData = await computeUserSummary(response?.data?.users || []);
        setDataInfo(userSummaryData);
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    };

    fetchData();
  }, []);

  return (
    <div className="App" style={{ padding: 20 }}>
      {Object.entries(dataInfo).map(([department, summary], index) => (
        <div key={index} style={{ padding: 20, border: '2px solid #eee', borderRadius: 10, marginBottom: 15 }}>
          <div>{index + 1}) Department: {department}</div>
          <div>male: {summary.male}</div>
          <div>female: {summary.female}</div>
          <div>ageRange: {summary.ageRange}</div>
          <div style={{ marginTop: 10 }}>hair:</div>
          <div style={{ padding: 10, border: '2px solid #eee', borderRadius: 5 }}>
            {Object.entries(summary.hair).map(([color, count], index) => (
              <div key={index}>{color}: {count}</div>
            ))}
          </div>
          <div style={{ marginTop: 10 }}>addressUser:</div>
          <div style={{ padding: 10, border: '2px solid #eee', borderRadius: 5 }}>
            {Object.entries(summary.addressUser).map(([name, postalCode], index) => (
              <div key={index}>{name}: {postalCode}</div>
            ))}
          </div>
        </div>
      ))}
    </div>
  );
}



